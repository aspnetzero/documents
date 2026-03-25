**Title:** How to Integrate Elsa v3 with ASP.NET Zero - Part 1: Backend Setup
**Description:** This blog walks through the backend setup for integrating Elsa v3 into an ASP.NET Zero solution. It covers installing required packages, configuring the Elsa server within the ASP.NET Zero architecture, setting up persistence, and preparing the application for workflow execution.

# How to Integrate Elsa v3 with ASP.NET Zero - Part 1: Backend Setup

## Introduction

[Elsa Workflows](https://v3.elsaworkflows.io/) is a powerful open-source workflow engine for .NET that enables you to build and execute workflows within your applications. With Elsa v3 integrated into ASP.NET Zero, the workflow engine has been adapted with custom enhancements to align with the ASP.NET Zero architecture, providing performance and experience alongside a modern Blazor-powered visual workflow designer.

[ASP.NET Zero](https://aspnetzero.com/) is an enterprise-grade application framework built on ASP.NET Core and Angular (React or MVC). It provides a solid foundation with features like multi-tenancy, authorization, audit logging, and much more out of the box.

In this two part series, we will walk you through integrating Elsa v3 into an ASP.NET Zero project. **Part 1** focuses on the backend setup, creating the Elsa Server project, configuring the workflow engine, sharing authentication, supporting multi-tenancy, and auto-launching the server. **Part 2** will cover the Angular frontend integration, where we embed the Elsa Studio designer into the ASP.NET Zero Angular UI.

### What You Will Build

By the end of this series, you will have:

- A standalone Elsa v3 workflow server running alongside your ASP.NET Zero host
- Shared JWT authentication between ASP.NET Zero and Elsa
- Full multi-tenancy support, each tenant gets isolated workflows
- An embedded Elsa Studio visual designer accessible from the ASP.NET Zero Angular UI
- A permission-based access control system for workflow management

### Prerequisites

- ASP.NET Zero project (Angular version) with .NET 10+
- SQL Server database
- Basic knowledge of ASP.NET Core, Entity Framework Core, and Blazor

## Architecture Overview

The integration follows a **sidecar architecture** the Elsa Server runs as a separate process alongside the ASP.NET Zero host, sharing the same database and JWT authentication configuration.

![Elsa v3 with ASP.NET Zero Architecture Diagram](/Images/Blog/elsa-v3-with-asp-net-zero-diagram.png)

### Key Architectural Decisions

| Decision | Rationale |
|---|---|
| **Separate process** | Elsa Server runs independently, avoiding dependency conflicts with ABP framework |
| **Shared database** | Elsa tables are auto-migrated into the same SQL Server database used by ASP.NET Zero |
| **Shared JWT tokens** | The same symmetric key, issuer, and audience are used by both servers, enabling seamless authentication |
| **Iframe embedding** | Elsa Studio (Blazor) is embedded in the Angular UI via an iframe with token passing |
| **ABP multi-tenancy mapping** | ABP tenant IDs are mapped to Elsa tenant IDs with a `tenant_` prefix |

### Projects Added to the Solution

| Project | Type | Purpose |
|---|---|---|
| `YourProjectName.ElsaServer` | ASP.NET Core Web App | Hosts the Elsa v3 engine + Blazor Server Elsa Studio |
| `YourProjectName.ElsaStudio` | Blazor WebAssembly | Optional standalone Elsa Studio client |

Now let's dive into the step-by-step implementation.

## Step 1: Create the ElsaServer Project

First, create a new ASP.NET Core Web Application project inside the `aspnet-core/src/` folder of your ASP.NET Zero solution.

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

Replace the contents of `YourProjectName.ElsaServer.csproj` with the following. This installs all the Elsa v3 core packages, Entity Framework Core persistence for SQL Server, scripting engines, multi-tenancy support, and the Blazor Server Elsa Studio UI:

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

> **Note:** It is important to add the `<RequiresAspNetWebAssets>true</RequiresAspNetWebAssets>` property in the PropertyGroup. Since the Elsa Studio packages include Blazor based static web assets (CSS, JavaScript, Monaco editor files, etc.), this property ensures that the ASP.NET Core shared framework's static web assets are properly resolved and served at runtime. Without it, you may encounter missing static file errors when loading the Elsa Studio UI.

Here is what each package group provides:

| Package Group | Packages | Purpose |
|---|---|---|
| **Core Engine** | `Elsa`, `Elsa.Workflows.Api` | The workflow engine core and REST API endpoints |
| **Persistence** | `Elsa.EntityFrameworkCore`, `Elsa.EntityFrameworkCore.SqlServer` | EF Core persistence to SQL Server |
| **Activities** | `Elsa.Http`, `Elsa.Scheduling` | HTTP trigger/response activities and scheduled workflows |
| **Scripting** | `Elsa.CSharp`, `Elsa.JavaScript`, `Elsa.Liquid` | C#, JavaScript, and Liquid expression support in workflows |
| **Identity** | `Elsa.Identity` | Elsa's identity module (we disable its built-in security) |
| **Multi-Tenancy** | `Elsa.Tenants` | Tenant isolation for workflows |
| **Studio UI** | `Elsa.Studio.*`, `Elsa.Api.Client` | Blazor Server visual designer and API client |
| **Auth** | `Microsoft.AspNetCore.Authentication.JwtBearer` | JWT Bearer authentication |
| **Data Access** | `Microsoft.Data.SqlClient` | Raw SQL access to read `AbpTenants` table |

### Configure Launch Settings

Create or modify `Properties/launchSettings.json` to set the development URLs:

```json
{
  "$schema": "https://json.schemastore.org/launchsettings.json",
  "profiles": {
    "YourProjectName.ElsaServer": {
      "commandName": "Project",
      "launchBrowser": true,
      "launchUrl": "",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "https://localhost:44311;http://localhost:44310"
    }
  }
}
```

The Elsa Server will run on **port 44311** (HTTPS), separate from the ASP.NET Zero host on port 44301.

### Create the Folder Structure

Create the following folder structure inside the `ElsaServer` project. Once we've created the necessary files, the structure will look like this:

```
YourProjectName.ElsaServer/
├── Middleware/
│   ├── ElsaApiAuthorizationMiddleware.cs
│   ├── TenantMiddleware.cs
│   ├── TokenProvider.cs
│   ├── TokenCircuitHandler.cs
│   └── AuthenticatingApiHttpMessageHandler.cs
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

We will create these files in the later steps of the blog post.

## Step 2: Configure the Elsa Workflow Engine

Now let's build the `Program.cs` file, the crucial of the Elsa Server. This file configures the entire workflow engine, persistence, scripting, and the Blazor Studio UI. We will build it incrementally.

### Disable Elsa's Built-in Security

Elsa v3 ships with its own API key-based security. Since we are using ASP.NET Zero's JWT authentication instead, we disable it at the very top of `Program.cs`:

```csharp
using Elsa;

// Disable Elsa's built-in security - we handle authentication via ASP.NET Zero JWT
EndpointSecurityOptions.DisableSecurity();
```

This is a static call that must happen before the builder is created.

### Set Up the Builder

```csharp
var builder = WebApplication.CreateBuilder(args);
var configuration = builder.Configuration;
var services = builder.Services;

builder.WebHost.UseStaticWebAssets();

// Add HttpContextAccessor for tenant resolution
services.AddHttpContextAccessor();
```

`UseStaticWebAssets()` is required so that the Blazor Server UI can serve its static files (CSS, JS, Monaco editor, etc.) correctly during development.

### Configure the Elsa Engine

This is where we register all Elsa modules. The engine uses a fluent configuration API:

```csharp
using Elsa.EntityFrameworkCore.Extensions;
using Elsa.EntityFrameworkCore.Modules.Management;
using Elsa.EntityFrameworkCore.Modules.Runtime;
using Elsa.Extensions;
using Elsa.Tenants.Extensions;

// Configure Elsa Workflow Engine
services.AddElsa(elsa =>
{
    // Workflow Management - stores workflow definitions
    elsa.UseWorkflowManagement(management => 
    {
        management.UseEntityFrameworkCore(ef => 
        {
            ef.UseSqlServer(configuration.GetConnectionString("Default"));
            ef.RunMigrations = true;
        });
    });

    // Workflow Runtime - stores workflow instances and execution state
    elsa.UseWorkflowRuntime(runtime => 
    {
        runtime.UseEntityFrameworkCore(ef => 
        {
            ef.UseSqlServer(configuration.GetConnectionString("Default"));
            ef.RunMigrations = true;
        });
    });

    // Expose Elsa REST API endpoints
    elsa.UseWorkflowsApi(api =>
    {
        api.AddFastEndpointsAssembly<Program>();
    });

    // Enable real-time workflow updates via SignalR
    elsa.UseRealTimeWorkflows();

    // Scripting engines for workflow expressions
    elsa.UseCSharp();
    elsa.UseJavaScript(options => options.AllowClrAccess = true);
    elsa.UseLiquid();

    // HTTP activities (HttpEndpoint trigger, SendHttpRequest, WriteHttpResponse)
    elsa.UseHttp(options =>
    {
        options.ConfigureHttpOptions = httpOptions =>
        {
            var baseUrl = configuration["Elsa:Http:BaseUrl"] ?? "https://localhost:44311";
            httpOptions.BaseUrl = new Uri(baseUrl);
            httpOptions.BasePath = "/workflows";
        };
    });

    // Scheduling (Timer, Cron, StartAt activities)
    elsa.UseScheduling();

    // Scan for custom activities and workflows in this assembly
    elsa.AddActivitiesFrom<Program>();
    elsa.AddWorkflowsFrom<Program>();

    // Enable multi-tenancy
    elsa.UseTenants();
});
```

Let's break down what each module does:

| Module | Purpose |
|---|---|
| `UseWorkflowManagement` | Manages workflow definitions (the blueprints). Stored in SQL Server via EF Core. |
| `UseWorkflowRuntime` | Manages workflow instances (running/completed workflows). Also stored in SQL Server. |
| `UseWorkflowsApi` | Exposes Elsa's REST API at `/elsa/api/*` using FastEndpoints. |
| `UseRealTimeWorkflows` | Enables SignalR-based real-time updates in the Studio UI. |
| `UseCSharp` / `UseJavaScript` / `UseLiquid` | Adds scripting engine support for workflow expressions. |
| `UseHttp` | Adds HTTP trigger and response activities. |
| `UseScheduling` | Adds timer, cron, and delay activities. |
| `AddActivitiesFrom<Program>` | Scans the assembly for custom activity classes. |
| `AddWorkflowsFrom<Program>` | Scans the assembly for programmatic workflow definitions. |
| `UseTenants` | Enables Elsa's multi-tenancy system. |

> **Important:** Setting `ef.RunMigrations = true` tells Elsa to automatically create its database tables in your existing SQL Server database. This means Elsa tables (prefixed with `Elsa`) will coexist alongside your ABP tables in the same database.

## Step 3: JWT Authentication Integration

The key to seamless integration is sharing the same JWT configuration between ASP.NET Zero and the Elsa Server. Both servers use the same **security key**, **issuer**, and **audience**, so a token issued by ASP.NET Zero is also valid on the Elsa Server.

Add the following JWT configuration to `Program.cs`, right after the builder setup and before `services.AddElsa(...)`:

```csharp
using System.Text;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;

// Get ASP.NET Zero JWT settings from configuration
var jwtSettings = configuration.GetSection("Authentication:JwtBearer");
var securityKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtSettings["SecurityKey"]!));

// Configure ASP.NET Zero compatible JWT authentication
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

    // Handle SignalR token from query string
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
            // Extract tenant information from claims for multi-tenancy support
            var tenantIdClaim = context.Principal?.FindFirst(
                "http://www.aspnetboilerplate.com/identity/claims/tenantId");
            if (tenantIdClaim != null && int.TryParse(tenantIdClaim.Value, out var tenantId))
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

// Elsa Workflow Config Here 
```

### How It Works

1. **Shared Security Key**: The `SecurityKey` in `appsettings.json` must match exactly between both servers. ASP.NET Zero uses this key to sign tokens, and the Elsa Server uses it to validate them.

2. **SignalR Token Handling**: Blazor Server uses SignalR for communication. Since WebSocket connections cannot send Authorization headers, the token is passed via the `access_token` query string parameter. The `OnMessageReceived` event extracts it for `/hubs` and `/_blazor` paths.

3. **Tenant Extraction**: When a token is validated, the `OnTokenValidated` event extracts the ABP tenant ID claim (`http://www.aspnetboilerplate.com/identity/claims/tenantId`) and stores it in `HttpContext.Items` for downstream middleware to use.

## Step 4: Multi-Tenancy Support

ASP.NET Zero uses integer-based tenant IDs, while Elsa v3 uses string-based tenant IDs. We need to bridge these two systems. This requires two components:

1. **`AspNetZeroTenantsProvider`** An `ITenantsProvider` implementation that reads tenants from the `AbpTenants` table
2. **`TenantMiddleware`** Middleware that extracts the tenant from the JWT and sets the Elsa tenant context

### AspNetZeroTenantsProvider

Create `YourProjectName.ElsaServer/MultiTenancy/AspNetZeroTenantsProvider.cs`:

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
                "Connection string 'Default' not found in configuration.");
        _logger = logger;
    }

    public static string ToElsaTenantId(int abpTenantId) => $"{TenantIdPrefix}{abpTenantId}";

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
        var tenants = new List<Tenant>();

        // Always include the host tenant
        tenants.Add(new Tenant
        {
            Id = HostTenantId,
            Name = "Host"
        });

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

            _logger.LogInformation(
                "Loaded {Count} tenants from AbpTenants table (including host)", 
                tenants.Count);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, 
                "Failed to load tenants from AbpTenants table. " +
                "Only host tenant will be available.");
        }

        return tenants;
    }

    public async Task<Tenant?> FindAsync(
        TenantFilter filter, CancellationToken cancellationToken = default)
    {
        if (filter.Id == HostTenantId)
        {
            return new Tenant { Id = HostTenantId, Name = "Host" };
        }

        var abpId = ToAbpTenantId(filter.Id);
        if (abpId == null)
            return null;

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
                var tenancyName = reader.GetString(1);
                var name = reader.GetString(2);

                return new Tenant
                {
                    Id = ToElsaTenantId(reader.GetInt32(0)),
                    Name = $"{name} ({tenancyName})"
                };
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, 
                "Failed to find tenant with ABP ID {TenantId}", abpId);
        }

        return null;
    }
}
```

This provider:
- Maps ABP tenant ID `1` to Elsa tenant ID `"tenant_1"`
- Maps the host (no tenant) to Elsa tenant ID `"host"`
- Reads directly from the `AbpTenants` table using raw SQL (via `Microsoft.Data.SqlClient`) to avoid ABP framework dependencies
- Only returns active, non-deleted tenants

### TenantMiddleware

Create `YourProjectName.ElsaServer/Middleware/TenantMiddleware.cs`:

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

            if (tenantIdClaim != null && 
                int.TryParse(tenantIdClaim.Value, out var abpTenantId))
            {
                elsaTenantId = AspNetZeroTenantsProvider.ToElsaTenantId(abpTenantId);
            }
            else
            {
                // No tenant claim means this is a host user
                elsaTenantId = AspNetZeroTenantsProvider.HostTenantId;
            }
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
            else
            {
                _logger.LogWarning(
                    "Tenant '{ElsaTenantId}' not found in provider", elsaTenantId);
            }
        }

        await _next(context);
    }
}

public static class TenantMiddlewareExtensions
{
    public static IApplicationBuilder UseTenantMiddleware(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<TenantMiddleware>();
    }
}
```

The middleware:
1. Checks if the user is authenticated
2. Reads the ABP tenant ID claim from the JWT token
3. Converts it to an Elsa tenant ID using the `AspNetZeroTenantsProvider.ToElsaTenantId` helper
4. Looks up the tenant from the provider
5. Pushes the tenant context using Elsa's `ITenantAccessor`, so all downstream Elsa operations are scoped to that tenant

### Register the Tenant Provider

In `Program.cs`, after `services.AddElsa(...)`, register the custom tenant provider:

```csharp
services.AddScoped<ITenantsProvider, AspNetZeroTenantsProvider>();
```

## Step 5: Blazor Server Elsa Studio Integration

The Elsa Server hosts both the workflow engine **and** the Elsa Studio visual designer as a Blazor Server application. This requires three supporting services to pass the JWT token from the initial HTTP request into the Blazor circuit, and then forward it to Elsa API calls.

### TokenProvider A Scoped Token Holder

Create `YourProjectName.ElsaServer/Middleware/TokenProvider.cs`. This is a simple scoped service that holds the JWT token for the current Blazor circuit:

```csharp
namespace YourProjectName.ElsaServer.Middleware;

public class TokenProvider
{
    public string? Token { get; set; }
}
```

### TokenCircuitHandler Capture Token on Circuit Init

Create `YourProjectName.ElsaServer/Middleware/TokenCircuitHandler.cs`. When a Blazor Server circuit is opened (a new SignalR connection from the browser), this handler captures the JWT token from the HTTP request and stores it in the scoped `TokenProvider`:

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
            // Try query string first (from iframe URL `token`, then SignalR `access_token`)
            var token = httpContext.Request.Query["token"].FirstOrDefault();
            if (string.IsNullOrEmpty(token))
            {
                token = httpContext.Request.Query["access_token"].FirstOrDefault();
            }

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
                    "Token captured for Blazor circuit {CircuitId} (token length: {Length})",
                    circuit.Id, token.Length);
            }
            else
            {
                _logger.LogWarning(
                    "No token found for Blazor circuit {CircuitId}. " +
                    "Elsa Studio API calls will be unauthenticated.", circuit.Id);
            }
        }

        return base.OnCircuitOpenedAsync(circuit, cancellationToken);
    }

    public override Task OnCircuitClosedAsync(
        Circuit circuit, CancellationToken cancellationToken)
    {
        _logger.LogDebug(
            "Blazor circuit {CircuitId} closed, token will be garbage collected " +
            "with the scope", circuit.Id);
        return base.OnCircuitClosedAsync(circuit, cancellationToken);
    }
}
```

### AuthenticatingApiHttpMessageHandler Forward Token to API

Create `YourProjectName.ElsaServer/Middleware/AuthenticatingApiHttpMessageHandler.cs`. When the Blazor Server Elsa Studio makes API calls to the Elsa engine (which runs in the same process), this handler adds the Bearer token from the `TokenProvider`:

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
        HttpRequestMessage request, 
        CancellationToken cancellationToken)
    {
        var serviceProvider = _blazorServiceAccessor.Services;
        var tokenProvider = serviceProvider?.GetService<TokenProvider>();
        var token = tokenProvider?.Token;

        if (!string.IsNullOrEmpty(token))
        {
            request.Headers.Authorization = 
                new AuthenticationHeaderValue("Bearer", token);
            _logger.LogDebug(
                "Added Bearer token to Elsa API request: {Url}", 
                request.RequestUri);
        }
        else
        {
            _logger.LogWarning(
                "No token available for Elsa API request: {Url}.", 
                request.RequestUri);
        }

        var response = await base.SendAsync(request, cancellationToken);

        if (response.StatusCode == System.Net.HttpStatusCode.Unauthorized)
        {
            _logger.LogWarning(
                "Elsa API returned 401 Unauthorized for: {Url}. " +
                "Token may be expired.", request.RequestUri);
        }

        return response;
    }
}
```

### Register Blazor Server & Elsa Studio in Program.cs

After registering the tenant provider, add the Blazor Server and Elsa Studio configuration:

```csharp
using Elsa.Studio.Core.BlazorServer.Extensions;
using Elsa.Studio.Dashboard.Extensions;
using Elsa.Studio.Extensions;
using Elsa.Studio.Models;
using Elsa.Studio.Shell.Extensions;
using Elsa.Studio.Workflows.Extensions;
using Elsa.Studio.Workflows.Designer.Extensions;
using Microsoft.AspNetCore.Components.Server.Circuits;

// Register Tenant Provider Config

// Blazor Server setup
services.AddRazorPages();
services.AddServerSideBlazor(options =>
{
    options.RootComponents.RegisterCustomElsaStudioElements();
});

// Elsa Studio shell and modules
services.AddCore();
services.AddShell();

// Configure Elsa Studio backend API connection
var backendApiConfig = new BackendApiConfig
{
    ConfigureBackendOptions = options =>
    {
        options.Url = new Uri(
            configuration["Elsa:Server:Url"] ?? "https://localhost:44311/elsa/api");
    },
    ConfigureHttpClientBuilder = options =>
    {
        options.AuthenticationHandler = typeof(AuthenticatingApiHttpMessageHandler);
    }
};
services.AddRemoteBackend(backendApiConfig);

// Add Elsa Studio modules
services.AddDashboardModule();
services.AddWorkflowsModule();

// Register our custom services
services.AddScoped<AuthenticatingApiHttpMessageHandler>();
services.AddScoped<TokenProvider>();
services.AddScoped<CircuitHandler, TokenCircuitHandler>();
```

### The _Host.cshtml Page

Create `YourProjectName.ElsaServer/Pages/_Host.cshtml` this is the Blazor Server host page. It contains JavaScript that handles token passing between the Angular iframe parent and the Blazor app:

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
    <title>Elsa Studio - ASP.NET Zero</title>
    
    <!-- Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" 
          rel="stylesheet" />
    <link href="https://fonts.googleapis.com/css2?family=Ubuntu:wght@300;400;500;700&display=swap" 
          rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;500;600;700&display=swap" 
          rel="stylesheet">
    
    <!-- MudBlazor CSS -->
    <link href="_content/MudBlazor/MudBlazor.min.css" rel="stylesheet" />
    <link href="_content/CodeBeam.MudBlazor.Extensions/MudExtensions.min.css" 
          rel="stylesheet" />
    
    <!-- Radzen CSS -->
    <link href="_content/Radzen.Blazor/css/material-base.css" rel="stylesheet">
    
    <!-- Elsa Studio CSS -->
    <link href="_content/Elsa.Studio.Shell/css/shell.css" rel="stylesheet">
    
    <link rel="icon" type="image/png" 
          href="_content/Elsa.Studio.Shell/favicon-32x32.png" />
    <component type="typeof(HeadOutlet)" render-mode="ServerPrerendered" />
</head>

<body>
    <component type="typeof(Elsa.Studio.Shell.App)" render-mode="ServerPrerendered" />
    
    <div id="blazor-error-ui">
        An unhandled error has occurred.
        <a href="" class="reload">Reload</a>
        <a class="dismiss">&#x1f5d9;</a>
    </div>
    
    <!-- Scripts -->
    <script src="_content/BlazorMonaco/jsInterop.js"></script>
    <script src="_content/BlazorMonaco/lib/monaco-editor/min/vs/loader.js"></script>
    <script src="_content/BlazorMonaco/lib/monaco-editor/min/vs/editor/editor.main.js"></script>
    <script src="_content/MudBlazor/MudBlazor.min.js"></script>
    <script src="_content/CodeBeam.MudBlazor.Extensions/MudExtensions.min.js"></script>
    <script src="_content/Radzen.Blazor/Radzen.Blazor.js"></script>
    
    <script>
        // Extract ASP.NET Zero token from URL query string or localStorage
        window.getAspNetZeroToken = function() {
            var urlParams = new URLSearchParams(window.location.search);
            var token = urlParams.get('token');
            
            if (token) {
                localStorage.setItem('elsa_aspnetzero_token', token);
                return token;
            }
            
            return localStorage.getItem('elsa_aspnetzero_token');
        };
        
        // Notify parent iframe that Elsa Studio is ready
        window.notifyElsaReady = function() {
            if (window.parent !== window) {
                window.parent.postMessage({ type: 'ELSA_STUDIO_READY' }, '*');
            }
        };

        // Listen for token updates from Angular parent
        window.addEventListener('message', function(event) {
            if (event.data && event.data.type === 'set-elsa-token' && event.data.token) {
                localStorage.setItem('elsa_aspnetzero_token', event.data.token);
            }
        });
    </script>
    
    <script src="_framework/blazor.server.js" autostart="false"></script>
    <script>
        (function() {
            var token = window.getAspNetZeroToken();

            Blazor.start({
                configureSignalR: function(builder) {
                    builder.withUrl('/_blazor', {
                        accessTokenFactory: function() {
                            return token || '';
                        }
                    });
                }
            }).then(function() {
                if (window.notifyElsaReady) {
                    window.notifyElsaReady();
                }
            });
        })();
    </script>
</body>

</html>
```

The key aspects of this page:

1. **Token from URL**: When the Angular UI embeds Elsa Studio in an iframe, it passes the JWT token as `?token=...` in the URL
2. **Token persistence**: The token is saved to `localStorage` so it survives page refreshes
3. **SignalR authentication**: `Blazor.start()` is called manually with `autostart="false"`, configuring the SignalR connection to include the token via `accessTokenFactory`
4. **Parent communication**: `postMessage` is used to notify the Angular parent when the Studio is ready, and to receive token updates

## Step 6: API Authorization Middleware

Since we disabled Elsa's built-in security, we need our own middleware to protect the Elsa REST API endpoints. Create `YourProjectName.ElsaServer/Middleware/ElsaApiAuthorizationMiddleware.cs`:

```csharp
namespace YourProjectName.ElsaServer.Middleware;

public class ElsaApiAuthorizationMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ElsaApiAuthorizationMiddleware> _logger;

    public ElsaApiAuthorizationMiddleware(
        RequestDelegate next, ILogger<ElsaApiAuthorizationMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var path = context.Request.Path.Value?.ToLowerInvariant() ?? "";
        
        // Check if this is an Elsa API request
        if (path.StartsWith("/elsa/api"))
        {
            // Require authentication for API endpoints
            if (context.User?.Identity?.IsAuthenticated != true)
            {
                _logger.LogWarning(
                    "Unauthorized access attempt to Elsa API: {Path}", path);
                context.Response.StatusCode = StatusCodes.Status401Unauthorized;
                await context.Response.WriteAsync(
                    "Authentication required for Elsa API");
                return;
            }
            
            _logger.LogDebug(
                "Authenticated user {User} accessing Elsa API: {Path}", 
                context.User.Identity?.Name ?? "unknown", path);
        }

        await _next(context);
    }
}

public static class ElsaApiAuthorizationMiddlewareExtensions
{
    public static IApplicationBuilder UseElsaApiAuthorization(
        this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<ElsaApiAuthorizationMiddleware>();
    }
}
```

This middleware intercepts all requests to `/elsa/api/*` and returns 401 if the user is not authenticated. Non-API requests (like Blazor static files) pass through normally.

## Step 7: ElsaServer Configuration & Middleware Pipeline

### appsettings.json

Create `appsettings.json` for the Elsa Server. The critical part is that the `Authentication:JwtBearer` section must match your ASP.NET Zero `appsettings.json` exactly:

```json
{
  "ConnectionStrings": {
    "Default": "Server=localhost; Database=YourProjectNameDb; Trusted_Connection=True; TrustServerCertificate=True;"
  },
  "App": {
    "CorsOrigins": "http://localhost:4200,https://localhost:44301,http://localhost:5173"
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
      "BaseUrl": "https://localhost:44311",
      "BasePath": "/workflows"
    },
    "Server": {
      "BaseUrl": "https://localhost:44311",
      "Url": "https://localhost:44311/elsa/api"
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

> **Important:** The `SecurityKey`, `Issuer`, and `Audience` values must be identical to those in your ASP.NET Zero `Web.Host/appsettings.json`. This is what allows tokens issued by ASP.NET Zero to be validated by the Elsa Server.

### CORS Configuration in Program.cs

Add CORS configuration to allow requests from the Angular UI and ASP.NET Zero host:

```csharp
services.AddCors(cors => cors
    .AddDefaultPolicy(policy => policy
        .WithOrigins(
            configuration["App:CorsOrigins"]?
                .Split(",", StringSplitOptions.RemoveEmptyEntries) 
            ?? ["http://localhost:4200", "https://localhost:44301", 
                "https://localhost:44311"]
        )
        .AllowAnyHeader()
        .AllowAnyMethod()
        .AllowCredentials()
        .WithExposedHeaders("x-elsa-workflow-instance-id")));

services.AddHealthChecks();
```

The `x-elsa-workflow-instance-id` header is exposed because Elsa uses it to return workflow instance IDs in HTTP responses.

### The Complete Middleware Pipeline

Finally, configure the middleware pipeline in `Program.cs`. The order matters:

```csharp
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();
app.UseCors();

// Authentication & Authorization
app.UseAuthentication();
app.UseAuthorization();

// Custom middleware
app.UseTenantMiddleware();         // Resolve ABP tenant → Elsa tenant
app.UseElsaApiAuthorization();     // Protect /elsa/api/* endpoints

// Elsa endpoints
app.UseWorkflowsApi();             // Mount /elsa/api/* REST endpoints
app.UseWorkflows();                // Enable workflow execution
app.UseWorkflowsSignalRHubs();     // SignalR hubs for real-time updates

// Blazor Server
app.MapBlazorHub();
app.MapFallbackToPage("/_Host");

// Health checks
app.MapHealthChecks("/health");

app.Run();
```

The middleware order is important:
1. **Authentication** runs first to validate JWT tokens
2. **TenantMiddleware** resolves the ABP tenant from the JWT and sets the Elsa tenant context
3. **ElsaApiAuthorization** protects API endpoints
4. **Elsa endpoints** handle workflow API requests and execution
5. **Blazor Server** serves the Studio UI for all other routes

## Step 8: Add Elsa Permissions to ASP.NET Zero

To control who can access Elsa workflows in the ASP.NET Zero UI, we define granular permissions. This requires changes to two existing files in the ASP.NET Zero solution.

### Define Permission Constants

Add the following constants to `YourProjectName.Core.Shared/Authorization/AppPermissions.cs`:

```csharp
//ELSA WORKFLOW PERMISSIONS
public const string Pages_Elsa = "Pages.Elsa";
public const string Pages_Elsa_WorkflowManagement = "Pages.Elsa.WorkflowManagement";
public const string Pages_Elsa_WorkflowDefinitions = "Pages.Elsa.WorkflowDefinitions";
public const string Pages_Elsa_WorkflowDefinitions_Create = "Pages.Elsa.WorkflowDefinitions.Create";
public const string Pages_Elsa_WorkflowDefinitions_Edit = "Pages.Elsa.WorkflowDefinitions.Edit";
public const string Pages_Elsa_WorkflowDefinitions_Delete = "Pages.Elsa.WorkflowDefinitions.Delete";
public const string Pages_Elsa_WorkflowInstances = "Pages.Elsa.WorkflowInstances";
public const string Pages_Elsa_WorkflowInstances_View = "Pages.Elsa.WorkflowInstances.View";
public const string Pages_Elsa_WorkflowInstances_Cancel = "Pages.Elsa.WorkflowInstances.Cancel";
public const string Pages_Elsa_WorkflowInstances_Delete = "Pages.Elsa.WorkflowInstances.Delete";
```

### Register the Permission Tree

Add the Elsa permission hierarchy in `YourProjectName.Core/Authorization/AppAuthorizationProvider.cs`, inside the `SetPermissions` method:

```csharp
// Elsa Workflow Permissions
var elsa = pages.CreateChildPermission(
    AppPermissions.Pages_Elsa, L("ElsaWorkflows"));
elsa.CreateChildPermission(
    AppPermissions.Pages_Elsa_WorkflowManagement, L("ElsaWorkflowManagement"));

var workflowDefinitions = elsa.CreateChildPermission(
    AppPermissions.Pages_Elsa_WorkflowDefinitions, L("ElsaWorkflowDefinitions"));
workflowDefinitions.CreateChildPermission(
    AppPermissions.Pages_Elsa_WorkflowDefinitions_Create, 
    L("ElsaWorkflowDefinitionsCreate"));
workflowDefinitions.CreateChildPermission(
    AppPermissions.Pages_Elsa_WorkflowDefinitions_Edit, 
    L("ElsaWorkflowDefinitionsEdit"));
workflowDefinitions.CreateChildPermission(
    AppPermissions.Pages_Elsa_WorkflowDefinitions_Delete, 
    L("ElsaWorkflowDefinitionsDelete"));

var workflowInstances = elsa.CreateChildPermission(
    AppPermissions.Pages_Elsa_WorkflowInstances, L("ElsaWorkflowInstances"));
workflowInstances.CreateChildPermission(
    AppPermissions.Pages_Elsa_WorkflowInstances_View, 
    L("ElsaWorkflowInstancesView"));
workflowInstances.CreateChildPermission(
    AppPermissions.Pages_Elsa_WorkflowInstances_Cancel, 
    L("ElsaWorkflowInstancesCancel"));
workflowInstances.CreateChildPermission(
    AppPermissions.Pages_Elsa_WorkflowInstances_Delete, 
    L("ElsaWorkflowInstancesDelete"));
```

This creates the following permission hierarchy:

```
Pages
└── Elsa (Pages.Elsa)
    ├── Workflow Management (Pages.Elsa.WorkflowManagement)
    ├── Workflow Definitions (Pages.Elsa.WorkflowDefinitions)
    │   ├── Create
    │   ├── Edit
    │   └── Delete
    └── Workflow Instances (Pages.Elsa.WorkflowInstances)
        ├── View
        ├── Cancel
        └── Delete
```

After adding these permissions, you can assign them to roles via the ASP.NET Zero admin UI. Only users with `Pages.Elsa` permission will see the Elsa menu item in the Angular UI (covered in Part 2).

## Step 9: Auto-Starting ElsaServer from Web.Host

Instead of manually running the Elsa Server as a separate process, we create a hosted service in the ASP.NET Zero `Web.Host` project that automatically launches it as a child process.

### ChildProcessHostedService Base Class

Create `YourProjectName.Web.Host/ElsaServer/ChildProcessHostedService.cs` in the `Web.Host` project. This is a reusable base class for launching any child .NET process:

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
            _logger.LogInformation(
                "{Name} auto-start is disabled.", options.Name);
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

        var workingDirectory = Path.GetDirectoryName(executablePath)!;

        _logger.LogInformation(
            "Starting {Name} from '{Path}'...", options.Name, executablePath);

        var startInfo = new ProcessStartInfo
        {
            FileName         = executablePath,
            WorkingDirectory = workingDirectory,
            UseShellExecute  = false,
            CreateNoWindow   = true,
            RedirectStandardOutput = true,
            RedirectStandardError  = true,
        };

        startInfo.Environment["ASPNETCORE_ENVIRONMENT"] = options.Environment;

        if (!string.IsNullOrWhiteSpace(options.Urls))
            startInfo.Environment["ASPNETCORE_URLS"] = options.Urls;

        _process = new Process 
        { 
            StartInfo = startInfo, 
            EnableRaisingEvents = true 
        };

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
            _logger.LogWarning(
                "{Name} process exited with code {Code}.", 
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
            _logger.LogError(ex, 
                "Failed to start {Name} process.", options.Name);
        }

        return Task.CompletedTask;
    }

    public async Task StopAsync(CancellationToken cancellationToken)
    {
        if (_process == null || _process.HasExited)
            return;

        var options = GetOptions();
        _logger.LogInformation(
            "Stopping {Name} (PID {Pid})...", options.Name, _process.Id);

        try
        {
            if (RuntimeInformation.IsOSPlatform(OSPlatform.Windows))
                _process.CloseMainWindow();
            else
                _process.Kill(entireProcessTree: false);

            await _process.WaitForExitAsync(cancellationToken);
            _logger.LogInformation("{Name} stopped.", options.Name);
        }
        catch (Exception ex)
        {
            _logger.LogWarning(ex, 
                "Error while stopping {Name}. Forcing kill.", options.Name);
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
        // If an explicit path is configured, use it
        if (!string.IsNullOrWhiteSpace(options.ExecutablePath))
            return options.ExecutablePath;

        // Otherwise, resolve relative to the host's build output
        var hostBinDir  = AppContext.BaseDirectory;
        var buildConfig = hostBinDir.Contains("Release", 
            StringComparison.OrdinalIgnoreCase) ? "Release" : "Debug";
        var framework   = $"net{Environment.Version.Major}.{Environment.Version.Minor}";

        return Path.GetFullPath(
            Path.Combine(hostBinDir,
                "..", "..", "..", "..",   // Navigate up from bin/Debug/net10.0/
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

The key feature of this base class is the `ResolveExecutablePath` method, it automatically finds the compiled executable of the child project relative to the host's build output. This means **you don't need to configure any paths** during development; it just works after building both projects.

### ElsaServerHostedService

Create `YourProjectName.Web.Host/ElsaServer/ElsaServerHostedService.cs`:

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
            Name           = "ElsaServer",
            ProjectName    = "YourProjectName.ElsaServer",
            ExecutablePath = section["ExecutablePath"],
            Environment    = section["Environment"] ?? "Development",
            AutoStart      = section.GetValue<bool>("AutoStart", defaultValue: true),
            Urls           = section["Urls"] 
                             ?? "https://localhost:44311;http://localhost:44310",
        };
    }
}
```

## Step 10: CORS & Web.Host Configuration Changes

With the hosted services and ElsaServer project in place, the final step is to wire everything together in the Web.Host project, adding configuration sections, registering the hosted services, and allowing cross-origin requests from the ElsaServer.

### Add ElsaServer Sections to appsettings.json

Open `YourProjectName.Web.Host/appsettings.json` and add the following two sections. These are read by the hosted services we created in the previous step:

```json
{
  "ElsaServer": {
    "AutoStart": true,
    "ExecutablePath": "",
    "Environment": "Development",
    "Urls": "https://localhost:44311;http://localhost:44310"
  }
}
```

Configuration options:

| Property | Description |
|---|---|
| `AutoStart` | Whether to automatically launch the child process when Web.Host starts. Set to `true` for ElsaServer.
| `ExecutablePath` | Leave empty for development, the hosted service auto-resolves the path relative to the build output. Set an absolute path for production deployments. |
| `Environment` | The `ASPNETCORE_ENVIRONMENT` passed to the child process. |
| `Urls` | The URLs the child process will listen on. Must match the URL configuration in the child project's `launchSettings.json`. |

### Add ElsaServer Origin to CORS

The ElsaServer's Blazor Studio makes API calls back to Web.Host for authentication token validation. You need to allow its origin in the CORS configuration. In `YourProjectName.Web.Host/appsettings.json`, add `https://localhost:44311` to the `CorsOrigins` list:

```json
{
  "App": {
    "CorsOrigins": "http://*.mycompany.com,http://localhost:4200,http://localhost:9876,http://localhost:5173,https://localhost:44311"
  }
}
```

The CORS policy in `Startup.cs` already reads from this comma-separated list and applies it dynamically, no code changes needed:

```csharp
services.AddCors(options =>
{
    options.AddPolicy(DefaultCorsPolicyName, builder =>
    {
        builder
            .WithOrigins(
                _appConfiguration["App:CorsOrigins"]
                    .Split(",", StringSplitOptions.RemoveEmptyEntries)
                    .Select(o => o.RemovePostFix("/"))
                    .ToArray()
            )
            .SetIsOriginAllowedToAllowWildcardSubdomains()
            .AllowAnyHeader()
            .AllowAnyMethod()
            .AllowCredentials();
    });
});
```

### Register the Hosted Services in Startup.cs

In `YourProjectName.Web.Host/Startup/Startup.cs`, add the `using` directive and register both hosted services in the `ConfigureServices` method:

```csharp
using YourProjectName.Web.ElsaServer;
```

Then, inside `ConfigureServices`, add these two lines (we placed them right after the `AddPasswordlessLoginRateLimit()` call):

```csharp
services.AddPasswordlessLoginRateLimit();

// Start Elsa Server as a child process when this host starts
services.AddHostedService<ElsaServerHostedService>();
```

That's it! When Web.Host starts, it will:

1. Read the `ElsaServer` section from `appsettings.json`
2. Since `AutoStart` is `true`, `ElsaServerHostedService` will resolve the executable path and launch the ElsaServer as a child process
3. The ElsaServer will start listening on `https://localhost:44311` with the Blazor Server Studio available at its root URL

### Verifying the Setup

Build the entire solution and run the Web.Host project:

```bash
dotnet build YourProjectName.Web.sln
dotnet run --project src/YourProjectName.Web.Host
```

You should see log output confirming the ElsaServer child process has started:

```
[ElsaServer] Starting child process...
[ElsaServer] Process started (PID: XXXXX)
[ElsaServer] Listening on https://localhost:44311
```

You can verify the setup by navigating to:

- **Web.Host API**: `https://localhost:44301/swagger` The ASP.NET Zero Swagger UI
- **Elsa Server API**: `https://localhost:44311/elsa/api` Elsa workflow API endpoints
- **Elsa Studio (Blazor Server)**: `https://localhost:44311` The embedded workflow designer

> **Important:** When you stop Web.Host (Ctrl+C), the `ChildProcessHostedService` base class will automatically terminate the ElsaServer child process via its `StopAsync` method.

## Summary

In this first part, we built the complete backend infrastructure for integrating the Elsa v3 workflow engine with an ASP.NET Zero application. Here's a recap of everything we accomplished:

1. **Created a standalone ElsaServer project**: A separate ASP.NET Core application that hosts the Elsa v3 workflow engine, configured with Entity Framework Core to store workflow data in the same database as our ASP.NET Zero application.

2. **Configured the Elsa engine**: Set up workflow runtime, JavaScript/Liquid scripting, HTTP activities, scheduling with Quartz, and EF Core persistence with automatic migrations.

3. **Shared JWT authentication**: Configured both the ElsaServer and Web.Host to use the same JWT security key, issuer, and audience, so tokens issued by ASP.NET Zero are accepted by the Elsa API.

4. **Added multi-tenancy support**: Built a custom `AspNetZeroTenantsProvider` that reads the ABP tenant table directly, mapping integer tenant IDs to Elsa's string-based tenant system with a `"tenant_"` prefix convention.

5. **Embedded Blazor Server Elsa Studio**: Integrated the Elsa Studio designer directly into the ElsaServer project, with a sophisticated token-passing mechanism: Angular passes the JWT via URL parameter → JavaScript stores it in localStorage → SignalR's `accessTokenFactory` passes it on each circuit → `TokenCircuitHandler` captures it → `AuthenticatingApiHttpMessageHandler` forwards it to Elsa API calls.

6. **Secured the Elsa API**: Created `ElsaApiAuthorizationMiddleware` that validates JWT tokens and ensures only authenticated users can access `/elsa/api/` endpoints.

7. **Configured the ElsaServer application**: Set up `appsettings.json` with shared connection strings and JWT settings, configured the middleware pipeline, and enabled CORS for the Angular frontend.

8. **Extended the permission system**: Added `Pages_Elsa` and its child permissions (workflow definitions, workflow instances) to `AppPermissions` and registered the permission tree in `AppAuthorizationProvider`.

9. **Auto-started ElsaServer from Web.Host**: Built a `ChildProcessHostedService` base class that manages child process lifecycle, with concrete `ElsaServerHostedService` implementations.

10. **Wired up CORS and configuration**: Added ElsaServer configuration sections to Web.Host's `appsettings.json`, registered the hosted services, and added the ElsaServer origin to the CORS policy.

### Architecture Recap

The final architecture looks like this:

- **Web.Host** (`https://localhost:44301`): The ASP.NET Zero backend API. When it starts, it automatically launches the ElsaServer as a child process.
- **ElsaServer** (`https://localhost:44311`): Hosts the Elsa v3 workflow engine API and the Blazor Server Elsa Studio designer. Shares the same database and JWT authentication with Web.Host.
- **Angular** (`http://localhost:4200`): The ASP.NET Zero frontend (covered in Part 2).

All three components share the same database and the same JWT tokens, providing a seamless single sign-on experience.

## What's Next Part 2: Angular Frontend Integration

In **Part 2** of this series, we'll complete the integration by embedding the Elsa Studio workflow designer into the ASP.NET Zero Angular application. Specifically, we'll cover:

- **Creating the Angular component**: An iframe-based component that loads the Blazor Server Elsa Studio, automatically passing the current user's JWT token via URL parameter.
- **Setting up routing**: Adding the Elsa Workflows page to the Angular router with proper permission guards.
- **Adding navigation**: Integrating the "Elsa Workflows" menu item into the ASP.NET Zero sidebar as a top-level entry consistent with the `/app/elsa/...` route.
- **Configuration**: Adding the ElsaServer URL to the Angular app's configuration so it can dynamically construct the iframe URL.

[How to Integrate Elsa v3 with ASP.NET Zero - Part 2: Angular Frontend Integration](https://aspnetzero.com/how-to-integrate-elsa-v3-with-aspnet-zero-part-2)

