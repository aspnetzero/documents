**Title:** How to Integrate Elsa v3 with ASP.NET Zero MVC
**Description:** This guide walks through integrating Elsa v3 into an ASP.NET Zero MVC project. It covers creating the ElsaServer sidecar process, configuring shared JWT authentication, multi-tenancy support, auto-starting the server, and embedding the Elsa Studio workflow designer inside an MVC Razor view via an iframe.

# How to Integrate Elsa v3 with ASP.NET Zero MVC

## Introduction

[Elsa Workflows](https://v3.elsaworkflows.io/) is a powerful open-source workflow engine for .NET that enables you to build and execute workflows within your applications. [ASP.NET Zero](https://aspnetzero.com/) is an enterprise-grade application framework built on ASP.NET Core and MVC. Combining the two gives you a fully integrated visual workflow designer accessible directly from your MVC application's sidebar menu.

This guide walks you through the complete integration. Unlike the Angular version (which is covered in a separate two-part series), the MVC integration is self-contained and covered entirely in this single article. By the end, you will have:

- A standalone Elsa v3 workflow server running as a child process alongside your ASP.NET Zero MVC host
- Shared JWT authentication between ASP.NET Zero and Elsa
- Full multi-tenancy support, each tenant gets isolated workflows
- An embedded Elsa Studio visual designer accessible from an MVC page

### Prerequisites

- ASP.NET Zero project (MVC version) with .NET 10+
- SQL Server database
- Basic knowledge of ASP.NET Core, Entity Framework Core

## Architecture Overview

The integration follows a **sidecar architecture**. The Elsa Server runs as a separate process alongside the ASP.NET Zero MVC host, sharing the same database and JWT authentication configuration.

![Elsa v3 with ASP.NET Zero Architecture Diagram](/Images/Blog/elsa-v3-with-asp-net-zero-diagram-mvc.png)

### Key Architectural Decisions

| Decision | Rationale |
|---|---|
| **Separate process** | Elsa Server runs independently, avoiding dependency conflicts with the ABP framework |
| **Shared database** | Elsa tables are auto-migrated into the same SQL Server database used by ASP.NET Zero |
| **Shared JWT tokens** | The same symmetric key, issuer, and audience are used by both servers, enabling seamless authentication |
| **Iframe embedding** | Elsa Studio (Blazor Server) is embedded in an MVC Razor view via an iframe with token passing |
| **ABP multi-tenancy mapping** | ABP tenant IDs are mapped to Elsa tenant IDs with a `tenant_` prefix |

### Projects Added to the Solution

| Project | Type | Purpose |
|---|---|---|
| `YourProjectName.ElsaServer` | ASP.NET Core Web App | Hosts the Elsa v3 engine + Blazor Server Elsa Studio |

## Step 1: Create the ElsaServer Project

Create a new ASP.NET Core Web Application project inside the `aspnet-core/src/` folder of your ASP.NET Zero solution.

```bash
cd aspnet-core/src
dotnet new web -n YourProjectName.ElsaServer -f net10.0
```

Then add the project to your solution:

```bash
cd ..
dotnet sln YourProjectName.Web.sln add src/YourProjectName.ElsaServer/YourProjectName.ElsaServer.csproj
```

### Install NuGet Packages

Replace the contents of `YourProjectName.ElsaServer.csproj` with the following:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <Import Project="..\..\common.props"></Import>
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <RootNamespace>YourProjectName.ElsaServer</RootNamespace>
    <AssemblyName>YourProjectName.ElsaServer</AssemblyName>
    <OutputType>Exe</OutputType>
    <RequiresAspNetWebAssets>true</RequiresAspNetWebAssets>
  </PropertyGroup>

  <ItemGroup>
    <!-- Elsa v3 Core Packages -->
    <PackageReference Include="Elsa" Version="3.5.3" />
    <PackageReference Include="Elsa.EntityFrameworkCore" Version="3.5.3" />
    <PackageReference Include="Elsa.EntityFrameworkCore.SqlServer" Version="3.5.3" />
    <PackageReference Include="Elsa.Http" Version="3.5.3" />
    <PackageReference Include="Elsa.Identity" Version="3.5.3" />
    <PackageReference Include="Elsa.Scheduling" Version="3.5.3" />
    <PackageReference Include="Elsa.Workflows.Api" Version="3.5.3" />
    <PackageReference Include="Elsa.CSharp" Version="3.5.3" />
    <PackageReference Include="Elsa.JavaScript" Version="3.5.3" />
    <PackageReference Include="Elsa.Liquid" Version="3.5.3" />

    <!-- Elsa Multi-Tenancy -->
    <PackageReference Include="Elsa.Tenants" Version="3.5.3" />

    <!-- Data Access for reading AbpTenants table -->
    <PackageReference Include="Microsoft.Data.SqlClient" Version="6.0.2" />

    <!-- Elsa Studio Packages for Server-Side Blazor -->
    <PackageReference Include="Elsa.Studio" Version="3.5.3" />
    <PackageReference Include="Elsa.Studio.Core.BlazorServer" Version="3.5.3" />
    <PackageReference Include="Elsa.Studio.Shell" Version="3.5.3" />
    <PackageReference Include="Elsa.Studio.Workflows" Version="3.5.3" />
    <PackageReference Include="Elsa.Studio.Workflows.Designer" Version="3.5.3" />
    <PackageReference Include="Elsa.Studio.Dashboard" Version="3.5.3" />
    <PackageReference Include="Elsa.Api.Client" Version="3.5.3" />

    <!-- JWT Authentication -->
    <PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="10.0.2" />
  </ItemGroup>
</Project>
```

> **Note:** The `<RequiresAspNetWebAssets>true</RequiresAspNetWebAssets>` property is required so that Blazor static web assets (CSS, JavaScript, Monaco editor files, etc.) are resolved and served correctly at runtime.

### Configure Launch Settings

Create `Properties/launchSettings.json`:

```json
{
  "$schema": "https://json.schemastore.org/launchsettings.json",
  "profiles": {
    "YourProjectName.ElsaServer": {
      "commandName": "Project",
      "launchBrowser": false,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "https://localhost:44313;http://localhost:44312"
    }
  }
}
```

The Elsa Server runs on **port 44313** (HTTPS), separate from the MVC host on port 44302.

### Create the Folder Structure

```
YourProjectName.ElsaServer/
├── Middleware/
│   ├── AuthenticatingApiHttpMessageHandler.cs
│   ├── ElsaApiAuthorizationMiddleware.cs
│   ├── TenantMiddleware.cs
│   ├── TokenCircuitHandler.cs
│   └── TokenProvider.cs
├── MultiTenancy/
│   └── AspNetZeroTenantsProvider.cs
├── Pages/
│   └── _Host.cshtml
├── Properties/
│   └── launchSettings.json
├── Program.cs
├── appsettings.json
└── YourProjectName.ElsaServer.csproj
```

## Step 2: Configure the Elsa Workflow Engine

### Disable Elsa's Built-in Security

Elsa v3 ships with its own API key-based security. Since we are using ASP.NET Zero's JWT authentication, we disable it at the top of `Program.cs`:

```csharp
using Elsa;

// Disable Elsa's built-in security - we handle authentication via ASP.NET Zero JWT
EndpointSecurityOptions.DisableSecurity();
```

This is a static call that must happen before the builder is created.

### Configure the Elsa Engine

```csharp
using Elsa.EntityFrameworkCore.Extensions;
using Elsa.EntityFrameworkCore.Modules.Management;
using Elsa.EntityFrameworkCore.Modules.Runtime;
using Elsa.Extensions;
using Elsa.Tenants.Extensions;

var builder = WebApplication.CreateBuilder(args);
var configuration = builder.Configuration;
var services = builder.Services;

builder.WebHost.UseStaticWebAssets();
services.AddHttpContextAccessor();

services.AddElsa(elsa =>
{
    elsa.UseWorkflowManagement(management =>
    {
        management.UseEntityFrameworkCore(ef =>
        {
            ef.UseSqlServer(configuration.GetConnectionString("Default"));
            ef.RunMigrations = true;
        });
    });

    elsa.UseWorkflowRuntime(runtime =>
    {
        runtime.UseEntityFrameworkCore(ef =>
        {
            ef.UseSqlServer(configuration.GetConnectionString("Default"));
            ef.RunMigrations = true;
        });
    });

    elsa.UseWorkflowsApi(api =>
    {
        api.AddFastEndpointsAssembly<Program>();
    });

    elsa.UseRealTimeWorkflows();
    elsa.UseCSharp();
    elsa.UseJavaScript(options => options.AllowClrAccess = true);
    elsa.UseLiquid();

    elsa.UseHttp(options =>
    {
        options.ConfigureHttpOptions = httpOptions =>
        {
            var baseUrl = configuration["Elsa:Http:BaseUrl"] ?? "https://localhost:44313";
            httpOptions.BaseUrl = new Uri(baseUrl);
            httpOptions.BasePath = "/workflows";
        };
    });

    elsa.UseScheduling();
    elsa.AddActivitiesFrom<Program>();
    elsa.AddWorkflowsFrom<Program>();
    elsa.UseTenants();
});
```

> **Important:** Setting `ef.RunMigrations = true` tells Elsa to automatically create its database tables in your existing SQL Server database on first run. Elsa tables (prefixed with `Elsa`) will coexist alongside your ABP tables in the same database. No separate migration step is needed.

## Step 3: JWT Authentication Integration

The key to seamless integration is sharing the same JWT configuration between ASP.NET Zero and the Elsa Server. Both servers use the same **security key**, **issuer**, and **audience**, so a token issued by ASP.NET Zero MVC is also valid on the Elsa Server.

Add the following to `Program.cs` before `services.AddElsa(...)`:

```csharp
using System.Text;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;

var jwtSettings = configuration.GetSection("Authentication:JwtBearer");
var securityKey = new SymmetricSecurityKey(
    Encoding.UTF8.GetBytes(jwtSettings["SecurityKey"]!));

services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = securityKey,
        ValidateIssuer = true,
        ValidIssuer = jwtSettings["Issuer"],
        ValidateAudience = true,
        ValidAudience = jwtSettings["Audience"],
        ValidateLifetime = true,
        ClockSkew = TimeSpan.Zero
    };

    // Handle SignalR/Blazor token from query string
    options.Events = new JwtBearerEvents
    {
        OnMessageReceived = context =>
        {
            var accessToken = context.Request.Query["access_token"];
            var path = context.HttpContext.Request.Path;

            if (!string.IsNullOrEmpty(accessToken) &&
                (path.StartsWithSegments("/hubs") || path.StartsWithSegments("/_blazor")))
            {
                context.Token = accessToken;
            }

            return Task.CompletedTask;
        },
        OnTokenValidated = context =>
        {
            var tenantIdClaim = context.Principal?.FindFirst(
                "http://www.aspnetboilerplate.com/identity/claims/tenantId");
            if (tenantIdClaim != null &&
                int.TryParse(tenantIdClaim.Value, out var tenantId))
            {
                context.HttpContext.Items["TenantId"] = tenantId;
            }
            return Task.CompletedTask;
        }
    };
});

services.AddAuthorization(options =>
{
    options.AddPolicy("ElsaPolicy", policy => policy.RequireAuthenticatedUser());
});
```

### How It Works

1. **Shared Security Key:** The `SecurityKey` in both `appsettings.json` files must match. ASP.NET Zero uses this key to sign tokens; the Elsa Server uses it to validate them.
2. **SignalR Token Handling:** Blazor Server communicates over SignalR. Since WebSocket connections cannot send Authorization headers, the token is passed via the `access_token` query string. The `OnMessageReceived` event extracts it for `/_blazor` paths.
3. **Tenant Extraction:** When a token is validated, the `OnTokenValidated` event extracts the ABP tenant ID claim and stores it in `HttpContext.Items` for the tenant middleware.

## Step 4: Multi-Tenancy Support

ASP.NET Zero uses integer-based tenant IDs while Elsa v3 uses string-based tenant IDs. We bridge these two systems with two components.

### AspNetZeroTenantsProvider

Create `MultiTenancy/AspNetZeroTenantsProvider.cs`:

```csharp
using Elsa.Common.Multitenancy;
using Microsoft.Data.SqlClient;

namespace YourProjectName.ElsaServer.MultiTenancy;

public class AspNetZeroTenantsProvider : ITenantsProvider
{
    private readonly string _connectionString;
    private readonly ILogger<AspNetZeroTenantsProvider> _logger;

    public const string TenantIdPrefix = "tenant_";
    public const string HostTenantId = "host";

    public AspNetZeroTenantsProvider(
        IConfiguration configuration,
        ILogger<AspNetZeroTenantsProvider> logger)
    {
        _connectionString = configuration.GetConnectionString("Default")
            ?? throw new InvalidOperationException(
                "Connection string 'Default' not found.");
        _logger = logger;
    }

    public static string ToElsaTenantId(int abpTenantId) =>
        $"{TenantIdPrefix}{abpTenantId}";

    public static int? ToAbpTenantId(string? elsaTenantId)
    {
        if (string.IsNullOrEmpty(elsaTenantId) || elsaTenantId == HostTenantId)
            return null;

        if (elsaTenantId.StartsWith(TenantIdPrefix) &&
            int.TryParse(elsaTenantId[TenantIdPrefix.Length..], out var id))
            return id;

        return null;
    }

    public async Task<IEnumerable<Tenant>> ListAsync(
        CancellationToken cancellationToken = default)
    {
        var tenants = new List<Tenant>
        {
            new Tenant { Id = HostTenantId, Name = "Host" }
        };

        try
        {
            await using var connection = new SqlConnection(_connectionString);
            await connection.OpenAsync(cancellationToken);

            const string sql = """
                SELECT Id, TenancyName, [Name]
                FROM AbpTenants
                WHERE IsDeleted = 0 AND IsActive = 1
                ORDER BY Id
                """;

            await using var command = new SqlCommand(sql, connection);
            await using var reader = await command.ExecuteReaderAsync(cancellationToken);

            while (await reader.ReadAsync(cancellationToken))
            {
                var abpId = reader.GetInt32(0);
                var tenancyName = reader.GetString(1);
                var name = reader.GetString(2);

                tenants.Add(new Tenant
                {
                    Id = ToElsaTenantId(abpId),
                    Name = $"{name} ({tenancyName})"
                });
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to load tenants from AbpTenants.");
        }

        return tenants;
    }

    public async Task<Tenant?> FindAsync(
        TenantFilter filter, CancellationToken cancellationToken = default)
    {
        if (filter.Id == HostTenantId)
            return new Tenant { Id = HostTenantId, Name = "Host" };

        var abpId = ToAbpTenantId(filter.Id);
        if (abpId == null) return null;

        try
        {
            await using var connection = new SqlConnection(_connectionString);
            await connection.OpenAsync(cancellationToken);

            const string sql = """
                SELECT Id, TenancyName, [Name]
                FROM AbpTenants
                WHERE Id = @Id AND IsDeleted = 0 AND IsActive = 1
                """;

            await using var command = new SqlCommand(sql, connection);
            command.Parameters.AddWithValue("@Id", abpId.Value);
            await using var reader = await command.ExecuteReaderAsync(cancellationToken);

            if (await reader.ReadAsync(cancellationToken))
            {
                return new Tenant
                {
                    Id = ToElsaTenantId(reader.GetInt32(0)),
                    Name = $"{reader.GetString(2)} ({reader.GetString(1)})"
                };
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to find tenant {TenantId}", abpId);
        }

        return null;
    }
}
```

This provider maps ABP tenant ID `1` → Elsa tenant ID `"tenant_1"`, and the host (no tenant) → `"host"`. It reads directly from the `AbpTenants` table using raw SQL to avoid ABP framework dependencies.

### TenantMiddleware

Create `Middleware/TenantMiddleware.cs`:

```csharp
using Elsa.Common.Multitenancy;
using YourProjectName.ElsaServer.MultiTenancy;

namespace YourProjectName.ElsaServer.Middleware;

public class TenantMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<TenantMiddleware> _logger;

    private const string AbpTenantIdClaimType =
        "http://www.aspnetboilerplate.com/identity/claims/tenantId";

    public TenantMiddleware(RequestDelegate next, ILogger<TenantMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(
        HttpContext context,
        ITenantAccessor tenantAccessor,
        ITenantsProvider tenantsProvider)
    {
        string? elsaTenantId = null;

        if (context.User?.Identity?.IsAuthenticated == true)
        {
            var tenantIdClaim = context.User.FindFirst(AbpTenantIdClaimType);

            elsaTenantId = tenantIdClaim != null &&
                           int.TryParse(tenantIdClaim.Value, out var abpTenantId)
                ? AspNetZeroTenantsProvider.ToElsaTenantId(abpTenantId)
                : AspNetZeroTenantsProvider.HostTenantId;
        }

        if (!string.IsNullOrEmpty(elsaTenantId))
        {
            var tenant = await tenantsProvider.FindAsync(
                TenantFilter.ById(elsaTenantId), context.RequestAborted);

            if (tenant != null)
            {
                using (tenantAccessor.PushContext(tenant))
                {
                    await _next(context);
                }
                return;
            }

            _logger.LogWarning("Tenant '{ElsaTenantId}' not found.", elsaTenantId);
        }

        await _next(context);
    }
}

public static class TenantMiddlewareExtensions
{
    public static IApplicationBuilder UseTenantMiddleware(
        this IApplicationBuilder builder) =>
        builder.UseMiddleware<TenantMiddleware>();
}
```

Register the provider in `Program.cs` after `services.AddElsa(...)`:

```csharp
services.AddScoped<ITenantsProvider, AspNetZeroTenantsProvider>();
```

## Step 5: Blazor Server Elsa Studio Integration

The Elsa Server hosts both the workflow engine and the Elsa Studio visual designer as a Blazor Server application. Three supporting services handle passing the JWT token from the initial HTTP request into the Blazor circuit.

### TokenProvider

Create `Middleware/TokenProvider.cs`. A simple scoped service that holds the JWT token for the current Blazor circuit:

```csharp
namespace YourProjectName.ElsaServer.Middleware;

public class TokenProvider
{
    public string? Token { get; set; }
}
```

### TokenCircuitHandler

Create `Middleware/TokenCircuitHandler.cs`. Captures the JWT token from the HTTP request when a Blazor Server circuit is opened:

```csharp
using Microsoft.AspNetCore.Components.Server.Circuits;

namespace YourProjectName.ElsaServer.Middleware;

public class TokenCircuitHandler : CircuitHandler
{
    private readonly IHttpContextAccessor _httpContextAccessor;
    private readonly TokenProvider _tokenProvider;
    private readonly ILogger<TokenCircuitHandler> _logger;

    public TokenCircuitHandler(
        IHttpContextAccessor httpContextAccessor,
        TokenProvider tokenProvider,
        ILogger<TokenCircuitHandler> logger)
    {
        _httpContextAccessor = httpContextAccessor;
        _tokenProvider = tokenProvider;
        _logger = logger;
    }

    public override Task OnCircuitOpenedAsync(
        Circuit circuit, CancellationToken cancellationToken)
    {
        var httpContext = _httpContextAccessor.HttpContext;

        if (httpContext != null)
        {
            // Try query string first (from iframe URL with ?access_token=...)
            var token = httpContext.Request.Query["access_token"].FirstOrDefault();

            // Fall back to Authorization header
            if (string.IsNullOrEmpty(token))
            {
                var authHeader = httpContext.Request.Headers.Authorization
                    .FirstOrDefault();
                if (!string.IsNullOrEmpty(authHeader) &&
                    authHeader.StartsWith("Bearer ", StringComparison.OrdinalIgnoreCase))
                {
                    token = authHeader["Bearer ".Length..];
                }
            }

            if (!string.IsNullOrEmpty(token))
            {
                _tokenProvider.Token = token;
                _logger.LogInformation(
                    "Token captured for Blazor circuit {CircuitId}.", circuit.Id);
            }
            else
            {
                _logger.LogWarning(
                    "No token found for Blazor circuit {CircuitId}.", circuit.Id);
            }
        }

        return base.OnCircuitOpenedAsync(circuit, cancellationToken);
    }
}
```

### AuthenticatingApiHttpMessageHandler

Create `Middleware/AuthenticatingApiHttpMessageHandler.cs`. Forwards the captured token on every Elsa API call made by the Blazor Studio:

```csharp
using System.Net.Http.Headers;
using Elsa.Studio.Contracts;

namespace YourProjectName.ElsaServer.Middleware;

public class AuthenticatingApiHttpMessageHandler : DelegatingHandler
{
    private readonly IBlazorServiceAccessor _blazorServiceAccessor;
    private readonly ILogger<AuthenticatingApiHttpMessageHandler> _logger;

    public AuthenticatingApiHttpMessageHandler(
        IBlazorServiceAccessor blazorServiceAccessor,
        ILogger<AuthenticatingApiHttpMessageHandler> logger)
    {
        _blazorServiceAccessor = blazorServiceAccessor;
        _logger = logger;
    }

    protected override async Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
    {
        var token = _blazorServiceAccessor.Services
            ?.GetService<TokenProvider>()?.Token;

        if (!string.IsNullOrEmpty(token))
        {
            request.Headers.Authorization =
                new AuthenticationHeaderValue("Bearer", token);
        }
        else
        {
            _logger.LogWarning(
                "No token for Elsa API request: {Url}", request.RequestUri);
        }

        var response = await base.SendAsync(request, cancellationToken);

        if (response.StatusCode == System.Net.HttpStatusCode.Unauthorized)
        {
            _logger.LogWarning(
                "Elsa API 401 for: {Url}. Token may be expired.", request.RequestUri);
        }

        return response;
    }
}
```

### Register Blazor Server & Elsa Studio

Add to `Program.cs`:

```csharp
using Elsa.Studio.Core.BlazorServer.Extensions;
using Elsa.Studio.Dashboard.Extensions;
using Elsa.Studio.Extensions;
using Elsa.Studio.Models;
using Elsa.Studio.Shell.Extensions;
using Elsa.Studio.Workflows.Extensions;
using Elsa.Studio.Workflows.Designer.Extensions;
using Microsoft.AspNetCore.Components.Server.Circuits;

services.AddRazorPages();
services.AddServerSideBlazor(options =>
{
    options.RootComponents.RegisterCustomElsaStudioElements();
});

services.AddCore();
services.AddShell();

var backendApiConfig = new BackendApiConfig
{
    ConfigureBackendOptions = options =>
    {
        options.Url = new Uri(
            configuration["Elsa:Server:Url"] ?? "https://localhost:44313/elsa/api");
    },
    ConfigureHttpClientBuilder = options =>
    {
        options.AuthenticationHandler = typeof(AuthenticatingApiHttpMessageHandler);
    }
};
services.AddRemoteBackend(backendApiConfig);

services.AddDashboardModule();
services.AddWorkflowsModule();

services.AddScoped<AuthenticatingApiHttpMessageHandler>();
services.AddScoped<TokenProvider>();
services.AddScoped<CircuitHandler, TokenCircuitHandler>();
```

### The _Host.cshtml Page

Create `Pages/_Host.cshtml`. The key difference from the Angular version is that the token is read from the `?access_token=` query parameter (the MVC controller appends it directly to the iframe URL):

```html
@page "/"
@using Elsa.Studio.Shell
@using Microsoft.AspNetCore.Components.Web
@namespace YourProjectName.ElsaServer.Pages
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <base href="~/" />
    <title>Elsa Studio</title>

    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" rel="stylesheet" />
    <link href="https://fonts.googleapis.com/css2?family=Ubuntu:wght@300;400;500;700&display=swap" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;500;600;700&display=swap" rel="stylesheet">

    <link href="_content/MudBlazor/MudBlazor.min.css" rel="stylesheet" />
    <link href="_content/CodeBeam.MudBlazor.Extensions/MudExtensions.min.css" rel="stylesheet" />
    <link href="_content/Radzen.Blazor/css/material-base.css" rel="stylesheet">
    <link href="_content/Elsa.Studio.Shell/css/shell.css" rel="stylesheet">
    <link rel="icon" type="image/png" href="_content/Elsa.Studio.Shell/favicon-32x32.png" />
    <component type="typeof(HeadOutlet)" render-mode="ServerPrerendered" />
</head>
<body>
    <component type="typeof(Elsa.Studio.Shell.App)" render-mode="ServerPrerendered" />

    <div id="blazor-error-ui">
        An unhandled error has occurred.
        <a href="" class="reload">Reload</a>
        <a class="dismiss">&#x1f5d9;</a>
    </div>

    <script src="_content/BlazorMonaco/jsInterop.js"></script>
    <script src="_content/BlazorMonaco/lib/monaco-editor/min/vs/loader.js"></script>
    <script src="_content/BlazorMonaco/lib/monaco-editor/min/vs/editor/editor.main.js"></script>
    <script src="_content/MudBlazor/MudBlazor.min.js"></script>
    <script src="_content/CodeBeam.MudBlazor.Extensions/MudExtensions.min.js"></script>
    <script src="_content/Radzen.Blazor/Radzen.Blazor.js"></script>

    <script>
        // Read the JWT token from the URL query string and persist it
        window.getAspNetZeroToken = function () {
            var urlParams = new URLSearchParams(window.location.search);
            var token = urlParams.get('access_token');
            if (token) {
                localStorage.setItem('elsa_aspnetzero_token', token);
                return token;
            }
            return localStorage.getItem('elsa_aspnetzero_token');
        };

        window.notifyElsaReady = function () {
            if (window.parent !== window) {
                window.parent.postMessage({ type: 'ELSA_STUDIO_READY' }, '*');
            }
        };
    </script>

    <script src="_framework/blazor.server.js" autostart="false"></script>
    <script>
        (function () {
            var token = window.getAspNetZeroToken();

            Blazor.start({
                configureSignalR: function (builder) {
                    builder.withUrl('/_blazor', {
                        accessTokenFactory: function () {
                            return token || '';
                        }
                    });
                }
            }).then(function () {
                if (window.notifyElsaReady) {
                    window.notifyElsaReady();
                }
            });
        })();
    </script>
</body>
</html>
```

The token flow in the MVC version is simpler than the Angular version:

1. The MVC controller generates a JWT and appends it to the iframe URL as `?access_token=<token>`
2. `_Host.cshtml` reads it from the query string on page load
3. It is stored in `localStorage` for persistence across Blazor reconnects
4. `Blazor.start()` is called with `accessTokenFactory` returning this token for the SignalR connection

## Step 6: API Authorization Middleware

Create `Middleware/ElsaApiAuthorizationMiddleware.cs` to protect the Elsa REST API endpoints:

```csharp
namespace YourProjectName.ElsaServer.Middleware;

public class ElsaApiAuthorizationMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ElsaApiAuthorizationMiddleware> _logger;

    public ElsaApiAuthorizationMiddleware(
        RequestDelegate next,
        ILogger<ElsaApiAuthorizationMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var path = context.Request.Path.Value?.ToLowerInvariant() ?? "";

        if (path.StartsWith("/elsa/api"))
        {
            if (context.User?.Identity?.IsAuthenticated != true)
            {
                _logger.LogWarning(
                    "Unauthorized access attempt to Elsa API: {Path}", path);
                context.Response.StatusCode = StatusCodes.Status401Unauthorized;
                await context.Response.WriteAsync("Authentication required.");
                return;
            }
        }

        await _next(context);
    }
}

public static class ElsaApiAuthorizationMiddlewareExtensions
{
    public static IApplicationBuilder UseElsaApiAuthorization(
        this IApplicationBuilder builder) =>
        builder.UseMiddleware<ElsaApiAuthorizationMiddleware>();
}
```

## Step 7: ElsaServer Configuration & Middleware Pipeline

### appsettings.json

The critical requirement is that `Authentication:JwtBearer` values must match your MVC project's `appsettings.json` exactly:

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost; Database=YourProjectNameDb; Trusted_Connection=True; TrustServerCertificate=True;"
  },
  "App": {
    "CorsOrigins": "https://localhost:44302,https://localhost:44313",
    "WebSiteRootAddress": "https://localhost:44302"
  },
  "Authentication": {
    "JwtBearer": {
      "IsEnabled": "true",
      "SecurityKey": "YourAspNetZeroProjectJWTSecurityKey",
      "Issuer": "YourProjectName",
      "Audience": "YourProjectName"
    }
  },
  "Elsa": {
    "Http": {
      "BaseUrl": "https://localhost:44313",
      "BasePath": "/workflows"
    },
    "Server": {
      "BaseUrl": "https://localhost:44313",
      "Url": "https://localhost:44313/elsa/api"
    }
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Elsa": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

> **Important:** The `SecurityKey`, `Issuer`, and `Audience` values must be identical to those in your MVC project's `appsettings.json`. This is what allows tokens issued by ASP.NET Zero MVC to be validated by the Elsa Server.

### CORS Configuration

```csharp
services.AddCors(cors => cors
    .AddDefaultPolicy(policy => policy
        .WithOrigins(
            configuration["App:CorsOrigins"]?
                .Split(",", StringSplitOptions.RemoveEmptyEntries)
            ?? ["https://localhost:44302", "https://localhost:44313"]
        )
        .AllowAnyHeader()
        .AllowAnyMethod()
        .AllowCredentials()
        .WithExposedHeaders("x-elsa-workflow-instance-id")));

services.AddHealthChecks();
```

### The Complete Middleware Pipeline

```csharp
var app = builder.Build();

if (app.Environment.IsDevelopment())
    app.UseDeveloperExceptionPage();
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

// Allow embedding in iframe from the MVC host origin
app.Use(async (context, next) =>
{
    var mvcOrigin = configuration["App:WebSiteRootAddress"]?.TrimEnd('/')
                    ?? "https://localhost:44302";
    context.Response.Headers.Remove("X-Frame-Options");
    context.Response.Headers.Append(
        "Content-Security-Policy",
        $"frame-ancestors 'self' {mvcOrigin}");
    await next();
});

app.UseRouting();
app.UseCors();

app.UseAuthentication();
app.UseAuthorization();

app.UseTenantMiddleware();
app.UseElsaApiAuthorization();

app.UseWorkflowsApi();
app.UseWorkflows();
app.UseWorkflowsSignalRHubs();

app.MapBlazorHub();
app.MapFallbackToPage("/_Host");
app.MapHealthChecks("/health");

app.Run();
```

> **MVC-specific middleware note:** The `frame-ancestors` CSP header middleware is unique to the MVC version. By default, browsers block cross-origin iframes. This middleware removes the default `X-Frame-Options` header and replaces it with a `Content-Security-Policy: frame-ancestors` directive that explicitly allows the MVC host origin to embed the Elsa Studio in an iframe.

The middleware order matters:
1. **Static files** served before routing
2. **CSP/frame-ancestors** set before any response is written
3. **Authentication** validates JWT tokens
4. **TenantMiddleware** maps ABP tenant → Elsa tenant context
5. **ElsaApiAuthorization** protects `/elsa/api/*` endpoints
6. **Elsa endpoints** handle workflow API and execution
7. **Blazor Server** serves the Studio UI for all other routes

## Step 8: Auto-Starting ElsaServer from Web.Mvc

Instead of starting the Elsa Server manually, the MVC host automatically launches it as a child process via a hosted service.

### ChildProcessHostedService Base Class

Create `ElsaServer/ChildProcessHostedService.cs` in the `Web.Mvc` project:

```csharp
using System;
using System.Diagnostics;
using System.IO;
using System.Runtime.InteropServices;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

namespace YourProjectName.Web.ElsaServer;

public abstract class ChildProcessHostedService : IDisposable
{
    private readonly ILogger _logger;
    private Process? _process;

    protected ChildProcessHostedService(ILogger logger)
    {
        _logger = logger;
    }

    protected abstract ChildProcessOptions GetOptions();

    public Task StartAsync(CancellationToken cancellationToken)
    {
        var options = GetOptions();

        if (!options.AutoStart)
        {
            _logger.LogInformation("{Name} auto-start is disabled.", options.Name);
            return Task.CompletedTask;
        }

        var executablePath = ResolveExecutablePath(options);

        if (string.IsNullOrWhiteSpace(executablePath) || !File.Exists(executablePath))
        {
            _logger.LogWarning(
                "{Name} executable not found at '{Path}'. " +
                "Build the project first or set ExecutablePath in appsettings.json.",
                options.Name, executablePath ?? "<not configured>");
            return Task.CompletedTask;
        }

        var startInfo = new ProcessStartInfo
        {
            FileName = executablePath,
            WorkingDirectory = Path.GetDirectoryName(executablePath)!,
            UseShellExecute = false,
            CreateNoWindow = true,
            RedirectStandardOutput = true,
            RedirectStandardError = true,
        };

        startInfo.Environment["ASPNETCORE_ENVIRONMENT"] = options.Environment;
        if (!string.IsNullOrWhiteSpace(options.Urls))
            startInfo.Environment["ASPNETCORE_URLS"] = options.Urls;

        _process = new Process { StartInfo = startInfo, EnableRaisingEvents = true };

        var name = options.Name;
        _process.OutputDataReceived += (_, e) =>
        {
            if (!string.IsNullOrEmpty(e.Data))
                _logger.LogInformation("[{Name}] {Line}", name, e.Data);
        };
        _process.ErrorDataReceived += (_, e) =>
        {
            if (!string.IsNullOrEmpty(e.Data))
                _logger.LogWarning("[{Name}] {Line}", name, e.Data);
        };
        _process.Exited += (_, _) =>
            _logger.LogWarning("{Name} process exited with code {Code}.",
                name, _process?.ExitCode);

        try
        {
            _process.Start();
            _process.BeginOutputReadLine();
            _process.BeginErrorReadLine();
            _logger.LogInformation(
                "{Name} started (PID {Pid}).", options.Name, _process.Id);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to start {Name} process.", options.Name);
        }

        return Task.CompletedTask;
    }

    public async Task StopAsync(CancellationToken cancellationToken)
    {
        if (_process == null || _process.HasExited) return;

        try
        {
            if (RuntimeInformation.IsOSPlatform(OSPlatform.Windows))
                _process.CloseMainWindow();
            else
                _process.Kill(entireProcessTree: false);

            await _process.WaitForExitAsync(cancellationToken);
        }
        catch (Exception ex)
        {
            _logger.LogWarning(ex, "Error stopping {Name}. Forcing kill.", GetOptions().Name);
            try { _process.Kill(entireProcessTree: true); } catch { }
        }
    }

    public void Dispose()
    {
        _process?.Dispose();
        _process = null;
    }

    private string? ResolveExecutablePath(ChildProcessOptions options)
    {
        if (!string.IsNullOrWhiteSpace(options.ExecutablePath))
            return options.ExecutablePath;

        var hostBinDir = AppContext.BaseDirectory;
        var buildConfig = hostBinDir.Contains("Release",
            StringComparison.OrdinalIgnoreCase) ? "Release" : "Debug";
        var framework = $"net{Environment.Version.Major}.{Environment.Version.Minor}";

        return Path.GetFullPath(
            Path.Combine(hostBinDir,
                "..", "..", "..", "..",
                options.ProjectName,
                "bin", buildConfig, framework,
                ExeName(options.ProjectName)));
    }

    private static string ExeName(string name) =>
        RuntimeInformation.IsOSPlatform(OSPlatform.Windows)
            ? $"{name}.exe" : name;
}

public sealed class ChildProcessOptions
{
    public string Name { get; init; } = "";
    public string ProjectName { get; init; } = "";
    public string? ExecutablePath { get; init; }
    public string Environment { get; init; } = "Development";
    public bool AutoStart { get; init; } = true;
    public string? Urls { get; init; }
}
```

`ResolveExecutablePath` navigates four levels up from the MVC host's `bin/Debug/net10.0/` directory to reach the `src/` folder, then constructs the path to the ElsaServer executable. This works automatically when both projects are siblings inside `src/`.

### ElsaServerHostedService

Create `ElsaServer/ElsaServerHostedService.cs` in the `Web.Mvc` project:

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;

namespace YourProjectName.Web.ElsaServer;

public class ElsaServerHostedService : ChildProcessHostedService, IHostedService
{
    private readonly IConfiguration _configuration;

    public ElsaServerHostedService(
        IConfiguration configuration,
        ILogger<ElsaServerHostedService> logger)
        : base(logger)
    {
        _configuration = configuration;
    }

    protected override ChildProcessOptions GetOptions()
    {
        var section = _configuration.GetSection("ElsaServer");
        return new ChildProcessOptions
        {
            Name = "ElsaServer",
            ProjectName = "YourProjectName.ElsaServer",
            ExecutablePath = section["ExecutablePath"],
            Environment = section["Environment"] ?? "Development",
            AutoStart = section.GetValue<bool>("AutoStart", defaultValue: true),
            Urls = section["Urls"] ?? "https://localhost:44313;http://localhost:44312",
        };
    }
}
```

### Register the Hosted Service

In `Web.Mvc/Startup/Startup.cs`, add the registration inside `ConfigureServices`:

```csharp
using YourProjectName.Web.ElsaServer;

// Inside ConfigureServices:
services.AddHostedService<ElsaServerHostedService>();
```

### Add ElsaServer Configuration

Add the `ElsaServer` section to `Web.Mvc/appsettings.json`:

```json
{
  "ElsaServer": {
    "AutoStart": true,
    "ExecutablePath": "",
    "Environment": "Development",
    "Urls": "https://localhost:44313;http://localhost:44312"
  }
}
```

Leave `ExecutablePath` empty to use the automatic path resolution. You can set it to an absolute path if you need to point to a specific binary (e.g., in a production deployment).

> **Before running:** You must build the ElsaServer project at least once so the executable exists:
> ```bash
> dotnet build src/YourProjectName.ElsaServer
> ```

## Step 9: Add the Elsa Workflows Page to the MVC UI

Now we wire everything into the MVC application: a page name constant, a menu item, a controller that generates the JWT, and a Razor view that renders the iframe.

### Add the Page Name Constant

In `Areas/App/Startup/AppPageNames.cs`, add `ElsaWorkflows` to the `Common` class:

```csharp
public static class Common
{
    // ... existing constants

    public const string ElsaWorkflows = "ElsaWorkflows";
}
```

### Add the Navigation Menu Item

In `Areas/App/Startup/AppNavigationProvider.cs`, add the menu item:

```csharp
.AddItem(new MenuItemDefinition(
    AppPageNames.Common.ElsaWorkflows,
    L("ElsaWorkflows"),
    url: "App/ElsaWorkflows",
    icon: "flaticon-squares-4"
))
```

### Add the Localization Key

In `Core/Localization/YourProjectName/YourProjectName.xml`:

```xml
<text name="ElsaWorkflows">Elsa Workflows</text>
```

### Create the Controller

Create `Areas/App/Controllers/ElsaWorkflowsController.cs`. This controller generates a short-lived JWT using the same signing key as the Elsa Server, then passes the iframe URL to the view:

```csharp
using System;
using System.Collections.Generic;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using Abp.AspNetCore.Mvc.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using YourProjectName.Web.Authentication.JwtBearer;
using YourProjectName.Web.Controllers;

namespace YourProjectName.Web.Areas.App.Controllers;

[Area("App")]
[AbpMvcAuthorize]
public class ElsaWorkflowsController : YourProjectNameControllerBase
{
    private readonly TokenAuthConfiguration _tokenAuthConfiguration;
    private readonly IConfiguration _configuration;

    public ElsaWorkflowsController(
        TokenAuthConfiguration tokenAuthConfiguration,
        IConfiguration configuration)
    {
        _tokenAuthConfiguration = tokenAuthConfiguration;
        _configuration = configuration;
    }

    public IActionResult Index()
    {
        var elsaServerUrl = _configuration["ElsaServer:Urls"]?
            .Split(";")[0]
            ?? "https://localhost:44313";

        var token = GenerateElsaToken();
        ViewBag.ElsaIframeUrl =
            $"{elsaServerUrl}?access_token={Uri.EscapeDataString(token)}";

        return View();
    }

    private string GenerateElsaToken()
    {
        var claims = new List<Claim>();

        if (AbpSession.UserId.HasValue)
        {
            claims.Add(new Claim(JwtRegisteredClaimNames.Sub,
                AbpSession.UserId.Value.ToString()));
            claims.Add(new Claim(JwtRegisteredClaimNames.NameId,
                AbpSession.UserId.Value.ToString()));
        }

        if (AbpSession.TenantId.HasValue)
        {
            claims.Add(new Claim(
                "http://www.aspnetboilerplate.com/identity/claims/tenantId",
                AbpSession.TenantId.Value.ToString()));
        }

        var nameClaim = User.FindFirst(ClaimTypes.Name)
                        ?? User.FindFirst(JwtRegisteredClaimNames.Name);
        if (nameClaim != null)
            claims.Add(new Claim(JwtRegisteredClaimNames.Name, nameClaim.Value));

        var emailClaim = User.FindFirst(ClaimTypes.Email)
                         ?? User.FindFirst(JwtRegisteredClaimNames.Email);
        if (emailClaim != null)
            claims.Add(new Claim(JwtRegisteredClaimNames.Email, emailClaim.Value));

        var now = DateTime.UtcNow;
        var jwt = new JwtSecurityToken(
            issuer: _tokenAuthConfiguration.Issuer,
            audience: _tokenAuthConfiguration.Audience,
            claims: claims,
            notBefore: now,
            signingCredentials: _tokenAuthConfiguration.SigningCredentials,
            expires: now.AddHours(8)
        );

        return new JwtSecurityTokenHandler().WriteToken(jwt);
    }
}
```

The controller uses `TokenAuthConfiguration` (already registered by ABP) to generate a JWT. Since both the MVC host and the Elsa Server share the same `SecurityKey`, `Issuer`, and `Audience`, the token generated here is valid for the Elsa Server's JWT middleware.

The token includes:
- `sub` / `nameid` — the ABP user ID
- The ABP tenant ID claim — used by the Elsa Server's `TenantMiddleware` for tenant isolation
- `name` and `email` — forwarded from the current MVC user principal

### Create the Razor View

Create `Areas/App/Views/ElsaWorkflows/Index.cshtml`:

```html
@{
    ViewBag.CurrentPageName =
        YourProjectName.Web.Areas.App.Startup.AppPageNames.Common.ElsaWorkflows;
}

<style>
    #elsa-studio-frame {
        width: 100%;
        height: calc(100vh - 65px);
        border: none;
        display: block;
    }
</style>

<div class="content d-flex flex-column flex-column-fluid p-0 m-0">
    <iframe id="elsa-studio-frame"
            src="@ViewBag.ElsaIframeUrl"
            allow="same-origin"
            title="Elsa Workflows">
    </iframe>
</div>
```

The iframe is set to `height: calc(100vh - 65px)` to fill the full height of the content area below the top navbar, and `width: 100%` to fill the area to the right of the sidebar. The result is a full-page Elsa Studio experience embedded within the MVC layout.

## Testing the Integration

### 1. Build the ElsaServer

The MVC host auto-starts the ElsaServer executable, so you must build it first:

```bash
dotnet build src/YourProjectName.ElsaServer
```

### 2. Start the MVC Application

```bash
dotnet run --project src/YourProjectName.Web.Mvc
```

You should see log output confirming both processes have started:

```
Web.Mvc listening on https://localhost:44302
[ElsaServer] Process started (PID: XXXXX)
[ElsaServer] Listening on https://localhost:44313
[ElsaServer] Running Elsa database migrations...
[ElsaServer] Migrations completed successfully.
```

### 3. Verify the Workflow

1. Open `https://localhost:44302` in your browser
2. Log in with an admin account
3. Look for **"Elsa Workflows"** in the sidebar navigation
4. Click it — the Elsa Studio should load inside the iframe
5. Try creating a workflow to verify the Elsa API is accessible

![Elsa v3 Workflows Page](/Images/Blog/elsa-v3-workflows-page.png)

### Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Sidebar item not visible | Localization key missing | Add `<text name="ElsaWorkflows">Elsa Workflows</text>` to the localization XML |
| "localhost refused to connect" | ElsaServer executable not found | Run `dotnet build src/YourProjectName.ElsaServer` first |
| Iframe shows blank / CSP error | Missing `frame-ancestors` header | Ensure the CSP middleware is in `Program.cs` and `App:WebSiteRootAddress` matches the MVC host URL |
| Blazor connection error (401) | JWT key mismatch | Verify `SecurityKey`, `Issuer`, and `Audience` match in both `appsettings.json` files |
| Workflows not isolated by tenant | Tenant middleware not resolving | Check that the ABP tenant ID claim (`http://www.aspnetboilerplate.com/identity/claims/tenantId`) is present in the generated JWT |

## Summary

In this guide, we integrated the **Elsa v3 workflow engine** into an **ASP.NET Zero MVC** application end-to-end:

1. **Created the `ElsaServer` project** with all required Elsa v3 packages and Blazor Server Studio
2. **Configured the workflow engine** with SQL Server persistence and automatic migrations
3. **Shared JWT authentication** by using the same security key, issuer, and audience as the MVC host
4. **Implemented multi-tenancy** by mapping ABP integer tenant IDs to Elsa string tenant IDs
5. **Built the Blazor token pipeline** (`TokenProvider` → `TokenCircuitHandler` → `AuthenticatingApiHttpMessageHandler`) so the Studio can authenticate its API calls
6. **Added the `frame-ancestors` CSP header** to allow the MVC host to embed the Blazor Studio in an iframe
7. **Auto-started the ElsaServer** as a child process via `ElsaServerHostedService`
8. **Added the MVC page** with a controller that generates a scoped JWT and a Razor view that renders the full-page iframe

The architecture keeps a clean separation of concerns: the Elsa Server runs independently and can be scaled or updated without touching the MVC application. From the user's perspective, the workflow designer feels like a native part of the MVC application.
