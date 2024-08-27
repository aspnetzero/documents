# Integrating ELSA with ASP.NET Zero

[Elsa](https://elsa-workflows.github.io/elsa-core/) is a great open source .NET Standard workflows library. Elsa allows creation of workflows by code, by JSON definition or by its workflow designer UI. In this document, we will basically integrate Elsa with ASP.NET Zero and create a simple workflow following official [Elsa Dashboard + Server](https://elsa-workflows.github.io/elsa-core/docs/next/quickstarts/quickstarts-aspnetcore-server-dashboard-and-api-endpoints) document.

Fully integrated Elsa sample project can be found on [GitHub](https://github.com/aspnetzero/aspnet-zero-samples/tree/master/ElsaDemo).

> Warning: In ASP.NET Zero versions prior to v13.0.0, Angular 16 is used. Elsa v2 is compatible with Angular 16, so it works seamlessly with these versions. However, when upgrading to newer versions of ASP.NET Zero, you should consider Elsa v2's compatibility.

## Create Project

In order to start integrating Elsa with ASP.NET Zero, first follow ASP.NET Zero's [Getting Started](Getting-Started-Core.md) document and create a new project. 

## Configuring Host Project

### Adding Elsa Packages

After creating the empty project, we need to add required Elsa NuGet packages to our project. Elsa document uses `Elsa.Persistence.EntityFramework.Sqlite` for persistence but we will use `Elsa.Persistence.EntityFramework.SqlServer` because ASP.NET Zero use SQL Server by default. So, add packages below to your `*.Web.Host` project.

* [Elsa](https://www.nuget.org/packages/Elsa)
* [Elsa.Activities.Http](https://www.nuget.org/packages/Elsa.Activities.Http)
* [Elsa.Activities.Temporal.Quartz](https://www.nuget.org/packages/Elsa.Activities.Temporal.Quartz)
* [Elsa.Persistence.EntityFramework.SqlServer](https://www.nuget.org/packages/Elsa.Persistence.EntityFramework.SqlServer)
* [Elsa.Server.Api](https://www.nuget.org/packages/Elsa.Server.Api)
* [Elsa.Server.Authentication](https://www.nuget.org/packages/Elsa.Server.Authentication)
* [Elsa.Designer.Components.Web](https://www.nuget.org/packages/Elsa.Designer.Components.Web)

### ConfigureServices Method

After adding Elsa NuGet packages, we need to configure Elsa in Startup.cs. So, open `Startup.cs` under `.Web.Host/Startup/` and configure it as shown below;

```c#
public IServiceProvider ConfigureServices(IServiceCollection services)
{
    ConfigureElsa(services);
    
    // Existing code blocks
}
```

```c#
private void ConfigureElsa(IServiceCollection services)
{
    services.AddElsa(elsa =>
        {
            elsa
            .UseEntityFrameworkPersistence(ef =>
                ef.UseSqlServer(_appConfiguration.GetConnectionString("Default"))
            )
            .AddConsoleActivities()
            .AddHttpActivities(_appConfiguration.GetSection("Elsa").GetSection("Server").Bind)
            .AddQuartzTemporalActivities()
            .AddJavaScriptActivities()
            .AddWorkflowsFrom<Startup>();
            
            elsa.UseAutoMapper(() => { });
        }
    );
    
    // Elsa API endpoints.
    services.AddZeroElsaApiEndpoints();
    services.Configure<ApiVersioningOptions>(options =>
    {
        options.UseApiBehavior = false;
    });
    
    services.Configure<RouteOptions>(options =>
    {
        options.LowercaseUrls = false;
    });

    // For Elsa Dashboard.
    services.AddRazorPages();
    
    // Remove ABP filter and add Elsa special filter
	services.PostConfigure<MvcOptions>(options =>
	{
    	ReplaceResultFilter(options);
	});
}
```

In this configuration, we are configuring Elsa to use existing ASP.NET Zero database and we configure Elsa with Default connection string. In order for this configuration to work, we need to add below section to `appsettings.json`.

````json
"Elsa": {
    "Server": {
      "BaseUrl": "https://localhost:44301"
    }
}
````

This will allow Elsa to use ASP.NET Zero application for handling HTTP activities.

There are some special configuration points here which doesn't exist in default Elsa document. Let's take a look at those sections one by one;

#### AutoMapper

````c#
elsa.UseAutoMapper(() => { });
````

Both Elsa and ASP.NET Boilerplate creates a `MapperConfiguration` and uses it to create an `IMapper`. Because of this, if we don't add this line to Elsa configuration, the app will only use Elsa's **AutoMapper** configuration. But, after making this configuration, we need to configure Elsa's AutoMapper mappings in our app manually. In order to do this, go to `*WebHostModule` and add below lines to its `PreInitialize` method.

```
// ELSA AutoMapper
Configuration.Modules.AbpAutoMapper().Configurators.Add(config =>
{
    // AutoMapperProfile
    config.CreateMap<IWorkflowBlueprint, WorkflowBlueprintModel>();
    config.CreateMap<IWorkflowBlueprint, WorkflowBlueprintSummaryModel>();
    config.CreateMap<IActivityBlueprint, ActivityBlueprintModel>().ConvertUsing<ActivityBlueprintConverter>();
    config.CreateMap<ICompositeActivityBlueprint, CompositeActivityBlueprintModel>();
    config.CreateMap<IConnection, ConnectionModel>().ConvertUsing<ConnectionConverter>();
    config.CreateMap<WorkflowInstance, WorkflowInstanceSummaryModel>();
    config.CreateMap<WorkflowDefinition, WorkflowDefinitionSummaryModel>();
    config.CreateMap<WorkflowDefinition, WorkflowDefinitionVersionModel>();
    
    // CloningProfile
    config.CreateMap<WorkflowDefinition, WorkflowDefinition>();
    config.CreateMap<WorkflowInstance, WorkflowInstance>();
});
```

#### API Versioning

```
services.AddElsaApiEndpoints();
services.Configure<ApiVersioningOptions>(options =>
{
    options.UseApiBehavior = false;
});

services.Configure<RouteOptions>(options =>
{
    options.LowercaseUrls = false;
});
```

ASP.NET Zero doesn't support API versioning and Elsa uses it by default. But, disabling `UseApiBehavior` makes ASP.NET Zero and Elsa to work together. `AddElsaApiEndpoints` configures `LowercaseUrls` of `RouteOptions` to true but ASP.NET Zero requires case sensitive URLs. So, we are converting `LowercaseUrls` of `RouteOptions` to false.

#### Result Filtering

````c#
services.PostConfigure<MvcOptions>(options =>
{
    ReplaceResultFilter(options);
});
````

ASP.NET Zero **wraps JSON** results by default and adds some additional fields to JSON result. The actual result is accessed by `result` property of the returned JSON result. But, Elsa doesn't work this way, so we should **ignore result wrapping** for Elsa's controllers. Unfortunately, ASP.NET Zero doesn't provide an easy configuration for doing that. Because of that, we need to replace default result filter with our own **custom result filter** using the configuration above. 

Here is the content of `ReplaceResultFilter` method;

```c#
private void ReplaceResultFilter(MvcOptions options)
{
    for (var index = options.Filters.Count - 1; index >= 0; --index)
    {
        if (options.Filters[index] is ServiceFilterAttribute &&
            ((ServiceFilterAttribute) options.Filters[index]).ServiceType != typeof(AbpResultFilter))
        {
            continue;
        }

        options.Filters.Insert(index, new ServiceFilterAttribute(typeof(ElsaResultFilter))
        {
            Order = 0
        });

        options.Filters.RemoveAt(index + 1);

        break;
    }
}
```

This method replaces `AbpResultFilter` with `ElsaResultFilter`. Here is the content of `ElsaResultFilter`;

```c#
using System.Linq;
using System.Reflection;
using Abp.AspNetCore.Configuration;
using Abp.AspNetCore.Mvc.Extensions;
using Abp.AspNetCore.Mvc.Results.Wrapping;
using Abp.Dependency;
using Abp.Reflection.Extensions;
using Elsa.Server.Api.Endpoints.Activities;
using Microsoft.AspNetCore.Mvc.Filters;

namespace ElsaAngularDemo.Web.Startup;

public class ElsaResultFilter : IResultFilter, ITransientDependency
{
    private readonly IAbpAspNetCoreConfiguration _configuration;
    private readonly IAbpActionResultWrapperFactory _actionResultWrapperFactory;

    public ElsaResultFilter(IAbpAspNetCoreConfiguration configuration, 
        IAbpActionResultWrapperFactory actionResultWrapper)
    {
        _configuration = configuration;
        _actionResultWrapperFactory = actionResultWrapper;
    }

    public virtual void OnResultExecuting(ResultExecutingContext context)
    {
        if (!context.ActionDescriptor.IsControllerAction())
        {
            return;
        }

        var controller = context.Controller;
        controller.GetType().GetAssembly();

        if (controller.GetType().Assembly == typeof(List).Assembly)
        {
            return;
        }
        
        var methodInfo = context.ActionDescriptor.GetMethodInfo();
        
        var wrapResultAttribute =
            GetSingleAttributeOfMemberOrDeclaringTypeOrDefault(
                methodInfo,
                _configuration.DefaultWrapResultAttribute
            );

        if (!wrapResultAttribute.WrapOnSuccess)
        {
            return;
        }

        _actionResultWrapperFactory.CreateFor(context).Wrap(context);
    }

    public void OnResultExecuted(ResultExecutedContext context)
    {
    }
    
    public TAttribute GetSingleAttributeOfMemberOrDeclaringTypeOrDefault<TAttribute>(MemberInfo memberInfo, TAttribute defaultValue = default(TAttribute), bool inherit = true)
        where TAttribute : class
    {
        return memberInfo.GetCustomAttributes(true).OfType<TAttribute>().FirstOrDefault()
               ?? memberInfo.ReflectedType?.GetTypeInfo().GetCustomAttributes(true).OfType<TAttribute>().FirstOrDefault()
               ?? defaultValue;
    }
}
```

### Configure Method

The additions of Configure method are shown in the code block below;

```c#
public void Configure(IApplicationBuilder app, IWebHostEnvironment env, ILoggerFactory loggerFactory)
{
    // existing code blocks

    app.UseStaticFiles();
    
    // ELSA
    app.UseHttpActivities();
    
    app.UseRouting();
    
    // existing code blocks
}
```

There is one additions here;

1. `app.UseHttpActivities();` this line allows Elsa to handle HTTP request and execute if there are any workflows related to HTTP activities.

### Elsa Controllers

By default Elsa controllers are not registered in ASP.NET Zero's dependency injection system. In order to do that, we must register Elsa Conrollers as shown below in the Initialize method of the Host Web Module;

```c#
public override void Initialize()
{
    // existing code blocks

    // Register controllers inside ELSA
    Register(typeof(Elsa.Server.Api.Endpoints.WorkflowDefinitions.List).GetAssembly());
    Register(typeof(Elsa.Server.Api.Endpoints.WorkflowDefinitions.Save).GetAssembly());
    Register(typeof(Elsa.Server.Authentication.Controllers.ElsaUserInfoController).GetAssembly());
}

private void Register(Assembly assembly)
{
    //Controller
    IocManager.IocContainer.Register(
        Classes.FromAssembly(assembly)
            .BasedOn<Controller>()
            .If(type => !type.GetTypeInfo().IsGenericTypeDefinition && !type.IsAbstract)
            .LifestyleTransient()
    );

    IocManager.IocContainer.Register(
        Classes.FromAssembly(assembly)
            .BasedOn<ControllerBase>()
            .If(type => !type.GetTypeInfo().IsGenericTypeDefinition && !type.IsAbstract)
            .LifestyleTransient()
    );

    //Razor Pages
    IocManager.IocContainer.Register(
        Classes.FromAssembly(assembly)
            .BasedOn<PageModel>()
            .If(type => !type.GetTypeInfo().IsGenericTypeDefinition && !type.IsAbstract)
            .LifestyleTransient()
    );

    //ViewComponents
    IocManager.IocContainer.Register(
        Classes.FromAssembly(assembly)
            .BasedOn<ViewComponent>()
            .If(type => !type.GetTypeInfo().IsGenericTypeDefinition)
            .LifestyleTransient()
    );
}
```

### UserManager

During the integration of Elsa, we faced an error of Serialization and Deserialization of `IdentityOptions` in UserManager's `InitializeOptionsAsync` method. To overcome this problem, you should override the related method in `UserManager.cs` as shown below;

```c#
public override async Task InitializeOptionsAsync(int? tenantId)
{
    Options = new IdentityOptions
    {
        Password =
        {
            RequiredUniqueChars = _optionsAccessor.Value.Password.RequiredUniqueChars
        },
        Stores =
        {
            ProtectPersonalData = _optionsAccessor.Value.Stores.ProtectPersonalData,
            MaxLengthForKeys = _optionsAccessor.Value.Stores.MaxLengthForKeys
        },
        Tokens =
        {
            AuthenticatorIssuer = _optionsAccessor.Value.Tokens.AuthenticatorIssuer,
            ProviderMap = _optionsAccessor.Value.Tokens.ProviderMap,
            AuthenticatorTokenProvider = _optionsAccessor.Value.Tokens.AuthenticatorTokenProvider,
            ChangeEmailTokenProvider = _optionsAccessor.Value.Tokens.ChangeEmailTokenProvider,
            EmailConfirmationTokenProvider = _optionsAccessor.Value.Tokens.EmailConfirmationTokenProvider,
            PasswordResetTokenProvider = _optionsAccessor.Value.Tokens.PasswordResetTokenProvider,
            ChangePhoneNumberTokenProvider = _optionsAccessor.Value.Tokens.ChangePhoneNumberTokenProvider
        },
        User =
        {
            RequireUniqueEmail = _optionsAccessor.Value.User.RequireUniqueEmail,
            AllowedUserNameCharacters = _optionsAccessor.Value.User.AllowedUserNameCharacters
        },
        ClaimsIdentity =
        {
            EmailClaimType = _optionsAccessor.Value.ClaimsIdentity.EmailClaimType,
            RoleClaimType = _optionsAccessor.Value.ClaimsIdentity.RoleClaimType,
            SecurityStampClaimType = _optionsAccessor.Value.ClaimsIdentity.SecurityStampClaimType,
            UserIdClaimType = _optionsAccessor.Value.ClaimsIdentity.UserIdClaimType,
            UserNameClaimType = _optionsAccessor.Value.ClaimsIdentity.UserNameClaimType
        },
        SignIn =
        {
            RequireConfirmedAccount = _optionsAccessor.Value.SignIn.RequireConfirmedAccount,
            RequireConfirmedEmail = _optionsAccessor.Value.SignIn.RequireConfirmedEmail,
            RequireConfirmedPhoneNumber = _optionsAccessor.Value.SignIn.RequireConfirmedPhoneNumber
        },
        Lockout =
        {
            AllowedForNewUsers = await IsTrueAsync(
                AbpZeroSettingNames.UserManagement.UserLockOut.IsEnabled,
                tenantId
            ),
            DefaultLockoutTimeSpan = TimeSpan.FromSeconds(
                await GetSettingValueAsync<int>(
                    AbpZeroSettingNames.UserManagement.UserLockOut.DefaultAccountLockoutSeconds,
                    tenantId
                )
            ),
            MaxFailedAccessAttempts = await GetSettingValueAsync<int>(
                AbpZeroSettingNames.UserManagement.UserLockOut.MaxFailedAccessAttemptsBeforeLockout,
                tenantId
            )
        }
    };
    
    // Password complexity
    Options.Password.RequireDigit = await GetSettingValueAsync<bool>(
        AbpZeroSettingNames.UserManagement.PasswordComplexity.RequireDigit,
        tenantId
    );

    Options.Password.RequireLowercase = await GetSettingValueAsync<bool>(
        AbpZeroSettingNames.UserManagement.PasswordComplexity.RequireLowercase,
        tenantId
    );

    Options.Password.RequireNonAlphanumeric = await GetSettingValueAsync<bool>(
        AbpZeroSettingNames.UserManagement.PasswordComplexity.RequireNonAlphanumeric,
        tenantId
    );

    Options.Password.RequireUppercase = await GetSettingValueAsync<bool>(
        AbpZeroSettingNames.UserManagement.PasswordComplexity.RequireUppercase,
        tenantId
    );

    Options.Password.RequiredLength = await GetSettingValueAsync<int>(
        AbpZeroSettingNames.UserManagement.PasswordComplexity.RequiredLength,
        tenantId
    );
}

private Task<bool> IsTrueAsync(string settingName, int? tenantId)
{
    return GetSettingValueAsync<bool>(settingName, tenantId);
}

private Task<T> GetSettingValueAsync<T>(string settingName, int? tenantId) where T : struct
{
    return tenantId == null
        ? _settingManager.GetSettingValueForApplicationAsync<T>(settingName)
        : _settingManager.GetSettingValueForTenantAsync<T>(settingName, tenantId.Value);
}
```

This approach manually creates a `IdentityOptions` and manually sets its fields one by one. The previous version uses **AutoMapper** which causes a problem.

### Hiding Elsa Controllers from Swagger

Since we are not using API Versioning in ASP.NET Zero, we must remove api-version parameter generated by adding ELSA to ASP.NET Zero. In order to do this, create `ZeroElsaServiceCollectionExtensions.cs` class as shown below next to Startup.cs.

```csharp
public static class ZeroElsaServiceCollectionExtensions
{
	public static IServiceCollection AddZeroElsaApiEndpoints(this IServiceCollection services,
		Action<ElsaApiOptions>? configureApiOptions = default)
	{
		var apiOptions = new ElsaApiOptions();
		configureApiOptions?.Invoke(apiOptions);

		var setupNewtonsoftJson = apiOptions.SetupNewtonsoftJson ?? (_ => { });

		services.AddControllers().AddNewtonsoftJson(setupNewtonsoftJson);
		services.AddRouting(options => { options.LowercaseUrls = true; });

		 // services.AddVersionedApiExplorer(o =>
		 // {
		 //     o.GroupNameFormat = "'v'VVV";
		 //     o.SubstituteApiVersionInUrl = false;
		 // });

		services.AddApiVersioning(
			options =>
			{
				options.UseApiBehavior = false;
				options.ReportApiVersions = false;
				options.DefaultApiVersion = ApiVersion.Default;
				options.AssumeDefaultVersionWhenUnspecified = true;
			});

		services
			.AddSingleton<ConnectionConverter>()
			.AddSingleton<ActivityBlueprintConverter>()
			.AddScoped<IWorkflowBlueprintMapper, WorkflowBlueprintMapper>()
			.AddSingleton<IEndpointContentSerializerSettingsProvider, EndpointContentSerializerSettingsProvider>()
			.AddAutoMapperProfile<AutoMapperProfile>()
			.AddSignalR();

		return services;
	}
}
```

Notice that `AddVersionedApiExplorer` call is commented out. Then, use this extension method `AddZeroElsaApiEndpoints` instead of `AddElsaApiEndpoints` in your Startup.cs;

```csharp
// Elsa API endpoints.
services.AddZeroElsaApiEndpoints();
services.Configure<ApiVersioningOptions>(options =>
{
    options.UseApiBehavior = false;
});
```

Then, go to `SwaggerEnumParameterFilter.cs` and add below if statement as the fist line to `Apply` method.

```csharp
public void Apply(OpenApiParameter parameter, ParameterFilterContext context)
{
	if (context.ApiParameterDescription == null || context.ApiParameterDescription.Type == null)
	{
		return;
	}
	
	// existing code blocks
}
```

Then, go to `SwaggerOperationFilter.cs` and change the line below;

```csharp
var enumType = context?.ApiDescription?.ParameterDescriptions[i]?.ParameterDescriptor?.ParameterType;
```

to this one;

```csharp
var enumType = context?.ApiDescription?.ParameterDescriptions[i]?.ParameterDescriptor?.ParameterType;
if (enumType == null)
{
	continue;
}
```

Finally, create a file named `SwaggerConfigExtensions.cs` in the Web.Core project under swagger folder and copy the content below;

```csharp
using Microsoft.Extensions.DependencyInjection;
using Swashbuckle.AspNetCore.SwaggerGen;

namespace Intelly.Web.Swagger
{
    public static class SwaggerConfigExtensions
    {
        public static void AddSwaggerRouteFilter(this SwaggerGenOptions options)
        {
            options.DocInclusionPredicate((docName, apiDesc) =>
            {
                var routeTemplate = apiDesc.RelativePath;

                if (routeTemplate == "workflow-definitions")
                    return false;
                else if (routeTemplate.Contains("workflow-definitions/import-files"))
                    return false;
                else if (routeTemplate.Contains("workflow-instances"))
                    return false;
                else if (routeTemplate.Contains("workflow-registry"))
                    return false;
                else if (routeTemplate.Contains("workflows"))
                    return false;
                else if (routeTemplate.Contains("v{apiVersion}"))
                    return false;
                else if (routeTemplate.Contains("v{version}"))
                    return false;

                return true;
            });
        }
    }
}
```

and, use this filter to hide Elsa endpoints in swagger document, so Swahsbuckle will not try to generate Typescript proxies for Elsa services. Add the change below in Startup.cs;

```csharp
services.AddSwaggerGen(options =>
{
	// existing configuration
	 options.AddSwaggerRouteFilter();
});
```

## Configuring Angular Project

### Adding NPM Packages

In order to use Elsa Dashboard on our Angular app, add [@elsa-workflows/elsa-workflows-studio](https://www.npmjs.com/package/@elsa-workflows/elsa-workflows-studio) NPM package to your Angular project.

To copy some of the scripts/styles we will use, add items below to `assets` section of the angular.json file;

````json
  {
    "glob": "**/*",
    "input": "node_modules/monaco-editor/min",
    "output": "./assets/monaco"
  },
  {
    "glob": "**/*",
    "input": "node_modules/@elsa-workflows/elsa-workflows-studio/dist/elsa-workflows-studio/assets",
    "output": "./assets/elsa-workflows-studio/"
  },
  {
    "glob": "*.css",
    "input": "node_modules/@elsa-workflows/elsa-workflows-studio/dist/elsa-workflows-studio",
    "output": "./assets/elsa-workflows-studio/"
  },
  {
    "glob": "*.js",
    "input": "node_modules/@elsa-workflows/elsa-workflows-studio/dist/elsa-workflows-studio",
    "output": "./assets/elsa-workflows-studio/"
  },
  {
    "glob": "*.png",
    "input": "node_modules/@elsa-workflows/elsa-workflows-studio/dist/elsa-workflows-studio/assets",
    "output": "./assets"
  },
````

Finally, add items below to index.html file to use Elsa Workflow Studio and Monaco Editor related style and script files;

````html
<link rel="icon" type="image/png" sizes="32x32" href="/assets/elsa-workflows-studio/images/favicon-32x32.png" />
<link rel="icon" type="image/png" sizes="16x16" href="/assets/elsa-workflows-studio/images/favicon-16x16.png" />
<link rel="stylesheet" href="/assets/elsa-workflows-studio/fonts/inter/inter.css" />
<link rel="stylesheet" href="/assets/elsa-workflows-studio/elsa-workflows-studio.css" />
````

### Creating Elsa Component

Before creating the Elsa component which will host Elsa Dashboard, let's create Elsa Module on our Angular app. To do this, create a folder named `elsa` under the `app\admin` folder.

Then, create the Elsa Module as shown below under the newly created `elsa` folder named `elsa.module.ts`.

````typescript
import { CUSTOM_ELEMENTS_SCHEMA, NgModule } from '@angular/core';
import { ElsaRoutingModule } from './elsa-routing.module';
import { AdminSharedModule } from '@app/admin/shared/admin-shared.module';
import { AppSharedModule } from '@app/shared/app-shared.module';
import { ElsaComponent } from './elsa.component';

@NgModule({
    declarations: [ElsaComponent],
    imports: [AppSharedModule, AdminSharedModule, ElsaRoutingModule],
    schemas: [CUSTOM_ELEMENTS_SCHEMA]
})
export class ElsaModule {}
````

Now, we can create `elsa.component.ts` as shown below;

````typescript
import { AfterViewChecked, Component, OnInit } from "@angular/core";
import { appModuleAnimation } from "@shared/animations/routerTransition";
import { AppComponentBase } from "@shared/common/app-component-base";
import { ScriptLoaderService } from '@shared/utils/script-loader.service';

@Component({
    templateUrl: './elsa.component.html',
    animations: [appModuleAnimation()],
})
export class ElsaComponent extends AppComponentBase implements OnInit, AfterViewChecked {
    
    ngOnInit(): void {
        new ScriptLoaderService().load('/assets/monaco/vs/loader.js');
    }

    ngAfterViewChecked(): void {
        const elsaStudioRoot = document.querySelector('elsa-studio-root');

        elsaStudioRoot.addEventListener('initializing', e => {
            const elsaStudio = (e as any).detail;
            elsaStudio.pluginManager.registerPlugin((window as any).AuthPlugin);
        });
    }
}
````

And, crate `elsa.component.html` as shown below;

````html
<div class="container-fluid">
	<div>
		<elsa-studio-root server-url="https://localhost:44301/" monaco-lib-path="assets/monaco">
			<elsa-studio-dashboard></elsa-studio-dashboard>
		</elsa-studio-root>
    </div>
</div>
````

Finally, create `elsa-routing.module.ts` to define routing for Elsa Dashboard;

````typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { ElsaComponent } from './elsa.component';

const routes: Routes = [
    {
        path: '',
        component: ElsaComponent,
        pathMatch: 'full',
    },
];

@NgModule({
    imports: [RouterModule.forChild(routes)],
    exports: [RouterModule],
})
export class ElsaRoutingModule {}
````

### Adding Elsa Component to Application

In order to use the Elsa component we have created, we need to add it to our Angular application. Since ASP.NET Zero lazy loads the modules, we need to add Elsa module to our Admin routing module. To do this, modify `admin-routing.module.ts` as shown below;

````typescript
{
	path: 'elsa',
	loadChildren: () => import('./elsa/elsa.module').then((m) => m.ElsaModule),
	data: { permission: 'Pages.Administration.Roles' },
},
````

Add line below to `typings.d.ts`;

````typescript
declare let monaco: any;
````

In order to use monaco editor in our angular app, add below items to `admin.module.ts`;

````typescript
// existing imports
import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';

@NgModule({
    imports: [
        // existing imports
    ],
    declarations: [],
    exports: [],
    providers: [
        existing providers
    ],
    ], 
	schemas: [CUSTOM_ELEMENTS_SCHEMA] // ELSA integration
})
export class AdminModule {}
````

Add `CUSTOM_ELEMENTS_SCHEMA` to `app.module.ts` as shown below;

````typescript
// existing imports
import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';

@NgModule({
    declarations: [
        // existing declerations
    ],
    providers: [
        // existing providers
    ],
    imports: [
        // existing imports
    ]
    ],
    schemas: [CUSTOM_ELEMENTS_SCHEMA]
})
export class AppModule {}
````

Finally, modify `main.ts` and lines below;

````typescript
import { defineCustomElements as elsaCustomElements } from '@elsa-workflows/elsa-workflows-studio/loader';

elsaCustomElements();
````

### Add Elsa Dashboard to Menu

In order to show Elsa on the main application menu, add item below to `app-navigation.service.ts`;

````typescript
new AppMenuItem('Workflows', null, 'flaticon2-setup', '/app/admin/elsa'),
````

### Authentication of Elsa Dsahboard

Elsa's Dashboard uses its own http client to call the server APIs. So, we need to send ASP.NET Zero's authentication token with Elsa's HTTP requests to server. In order to do this, we will be using Elsa Studio's custom plugin approach. For more info, you can check its [documentation](https://elsa-workflows.github.io/elsa-core/docs/next/extensibility/extensibility-designer-plugins#custom-plugins).

So, we need to add code block below to `index.html` file;

````javascript
<script>
	window.AuthPlugin = function (elsaStudio) {
		const { eventBus } = elsaStudio;

		const getAccessToken = async () => {
			return abp.auth.getToken();
		};

		const configureAuthMiddleware = async (e) => {
			const token = await getAccessToken();

			if (!token) return;

			e.service.register({
				onRequest(request) {
					request.headers = { Authorization: `Bearer ${token}` };
					if(request.method === 'post'){
						request.headers['Content-Type'] = 'application/json';
					}
					return request;
				},
			});
		};

		// Handle the "http-client-created" event so we con configure the http client.
		eventBus.on('http-client-created', configureAuthMiddleware);
	};
</script>
````

### Styling

There is a minor style problem when using Elsa Dashboard in ASP.NET Zero's Angular app. In order to fix this problem, add style below to `layout.less` file;

````less
elsa-modal-dialog .elsa-fixed {
    margin-top: 120px !important;
}
````

That's all. You can now run the application and navigate to Elsa Dashboard page by clicking the Elsa menu item.

## Sample Workflow

You can create a sample workflow as explained in [Elsa Documentation](https://elsa-workflows.github.io/elsa-core/docs/next/quickstarts/quickstarts-aspnetcore-server-dashboard-and-api-endpoints#the-workflow). This sample workflow starts with a request to `/hello-world` endpoint and returns the specified HTTP response.