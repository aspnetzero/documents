# Integrating ELSA with ASP.NET Zero

[Elsa](https://elsa-workflows.github.io/elsa-core/) is a great open source .NET Standard workflows library. Elsa allows creation of workflows by code, by JSON definition or by its workflow designer UI. In this document, we will basically integrate Elsa with ASP.NET Zero and create a simple workflow following official [Elsa Dashboard + Server](https://elsa-workflows.github.io/elsa-core/docs/next/quickstarts/quickstarts-aspnetcore-server-dashboard-and-api-endpoints) document.

Fully integrated Elsa sample project can be found on [GitHub](https://github.com/aspnetzero/aspnet-zero-samples/tree/master/ElsaDemo).

## Create Project

In order to start integrating Elsa with ASP.NET Zero, first follow ASP.NET Zero's [Getting Started](Getting-Started-Core.md) document and create a new project. 

## Adding Elsa Packages

After creating the empty project, we need to add required Elsa NuGet packages to our project. Elsa document uses ```Elsa.Persistence.EntityFramework.Sqlite``` for persistence but we will use ```Elsa.Persistence.EntityFramework.SqlServer``` because ASP.NET Zero use SQL Server by default. So, add packages below to your *.Web.Mvc project.

* [Elsa](https://www.nuget.org/packages/Elsa)
* [Elsa.Activities.Http](https://www.nuget.org/packages/Elsa.Activities.Http)
* [Elsa.Activities.Temporal.Quartz](https://www.nuget.org/packages/Elsa.Activities.Temporal.Quartz)
* [Elsa.Persistence.EntityFramework.SqlServer](https://www.nuget.org/packages/Elsa.Persistence.EntityFramework.SqlServer)
* [Elsa.Server.Api](https://www.nuget.org/packages/Elsa.Server.Api)
* [Elsa.Designer.Components.Web](https://www.nuget.org/packages/Elsa.Designer.Components.Web)

## Configuring Elsa

### ConfigureServices Method

After adding Elsa NuGet packages, we need to configure Elsa in Startup.cs. So, open ```Startup.cs``` under ***.Web.Mvc/Startup/** and configure it as shown below;

````c#
public IServiceProvider ConfigureServices(IServiceCollection services)
{
    ConfigureElsa(services);
    
    // Existing code blocks
}
````

````c#
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
    services.AddElsaApiEndpoints();
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
````

In this configuration, we are configuring Elsa to use existing ASP.NET Zero database and we configure Elsa with Default connection string. In order for this configuration to work, we need to add below section to **appsettings.json**.

````json
"Elsa": {
    "Server": {
      "BaseUrl": "https://localhost:44302"
    }
}
````

This will allow Elsa to use ASP.NET Zero application for handling HTTP activities.

There are some special configuration points here which doesn't exist in default Elsa document. Let's take a look at those sections one by one;

#### AutoMapper

````c#
elsa.UseAutoMapper(() => { });
````

Both Elsa and ASP.NET Boilerplate creates a ```MapperConfiguration``` and uses it to create an `IMapper`. Because of this, if we don't add this line to Elsa configuration, the app will only use Elsa's AutoMapper configuration. But, after making this configuration, we need to configure Elsa's AutoMapper mappings in our app manually. In order to do this, go to ***WebMvcModule** and add below lines to its ```PreInitialize``` method.

````
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
    
    // CloningProfile
    config.CreateMap<WorkflowDefinition, WorkflowDefinition>();
    config.CreateMap<WorkflowInstance, WorkflowInstance>();
});
````

#### API Versioning

````
services.AddElsaApiEndpoints();
services.Configure<ApiVersioningOptions>(options =>
{
    options.UseApiBehavior = false;
});

services.Configure<RouteOptions>(options =>
{
    options.LowercaseUrls = false;
});
````

ASP.NET Zero doesn't support API versioning and Elsa uses it by default. But, disabling ```UseApiBehavior``` makes ASP.NET Zero and Elsa to work together. ```AddElsaApiEndpoints``` configures ```LowercaseUrls``` of ```RouteOptions``` to true but ASP.NET Zero requires case sensitive URLs. So, we are converting ```LowercaseUrls``` of ```RouteOptions``` to false.

#### Result Filtering

````c#
services.PostConfigure<MvcOptions>(options =>
{
    ReplaceResultFilter(options);
});
````

ASP.NET Zero wraps JSON results by default and adds some additional fields to JSON result. The actual result is accessed by ```result ``` property of the returned JSON result. But, Elsa doesn't work this way, so we should ignore result wrapping for Elsa's controllers. Unfortunately, ASP.NET Zero doesn't provide an easy configuration for doing that. Because of that, we need to replace default result filter with our own custom result filter using the configuration above. 

Here is the content of ```ReplaceResultFilter``` method;

````c#
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
````

This method replaces ```AbpResultFilter``` with ```ElsaResultFilter```. Here is the content of ```ElsaResultFilter```;

````c#
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
````

### Configure Method

The additions of Configure method are shown in the code block below;

````c#
public void Configure(IApplicationBuilder app, IWebHostEnvironment env, ILoggerFactory loggerFactory)
{
    // existing code blocks

    app.UseStaticFiles();
    
    // ELSA
    app.UseHttpActivities();
    
    app.UseRouting();
    
    // existing code blocks
    
    app.UseEndpoints(endpoints =>
    {
        // existing code blocks
        
        // For Dashboard.
        endpoints.MapFallbackToPage("/_Host");
    });
    
    // existing code blocks
}
````

There are two additions here;

1. ```app.UseHttpActivities();``` this line allows Elsa to handle HTTP request and execute if there are any workflows related to HTTP activities.
2. ```endpoints.MapFallbackToPage("/_Host");``` this line redirects all not found requests to _Host.cshtml page which we will create in the next section. This page will contain Elsa Dashboard source code.

### Elsa Controllers

By default Elsa controllers are not registered in ASP.NET Zero's dependency injection system. In order to do that, we must register Elsa Conrollers as shown below in the Initialize method of the MVC Wed Module;

````c#
public override void Initialize()
{
    // existing code blocks

    // Register controllers inside ELSA
    Register(typeof(Elsa.Server.Api.Endpoints.WebhookDefinitions.List).GetAssembly());
    Register(typeof(Elsa.Server.Api.Endpoints.WorkflowDefinitions.Save).GetAssembly());
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
````

## Elsa Dashboard

In order to add Elsa Dashboard into ASP.NET Zero, create a folder named **Pages** under the MVC project and create a razor page named ```_Host.cshtml``` with the content below;

````html
@page "/"
@{
    var serverUrl = $"{Request.Scheme}://{Request.Host}";
}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>Elsa Workflows</title>
    <link rel="icon" type="image/png" sizes="32x32" href="/_content/Elsa.Designer.Components.Web/elsa-workflows-studio/assets/images/favicon-32x32.png">
    <link rel="icon" type="image/png" sizes="16x16" href="/_content/Elsa.Designer.Components.Web/elsa-workflows-studio/assets/images/favicon-16x16.png">
    <link rel="stylesheet" href="/_content/Elsa.Designer.Components.Web/elsa-workflows-studio/assets/fonts/inter/inter.css">
    <link rel="stylesheet" href="/_content/Elsa.Designer.Components.Web/elsa-workflows-studio/assets/styles/tailwind.css">
    <script src="/_content/Elsa.Designer.Components.Web/monaco-editor/min/vs/loader.js"></script>
    <script type="module" src="/_content/Elsa.Designer.Components.Web/elsa-workflows-studio/elsa-workflows-studio.esm.js"></script>
</head>
<body class="h-screen" style="background-size: 30px 30px; background-image: url(/_content/Elsa.Designer.Components.Web/elsa-workflows-studio/assets/images/tile.png); background-color: #FBFBFB;">
<elsa-studio-root server-url="@serverUrl" monaco-lib-path="_content/Elsa.Designer.Components.Web/monaco-editor/min"></elsa-studio-root>
</body>
</html>
````

ASP.NET Zero uses ``` WebHostBuilder``` in its Program.cs but it has some problems with embedded static files. Elsa provides styles and scripts of its dashboard as static files. In order to load Elsa's static files, change the ```Program.cs``` as shown below;

````c#
public class Program
{
    public static void Main(string[] args)
    {
        CurrentDirectoryHelpers.SetCurrentDirectory();
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args)
    {
        return WebHost.CreateDefaultBuilder()
            .UseKestrel(opt =>
            {
                opt.AddServerHeader = false;
                opt.Limits.MaxRequestLineSize = 16 * 1024;
            })
            .UseContentRoot(Directory.GetCurrentDirectory())
            .UseIIS()
            .UseIISIntegration()
            .UseStartup<Startup>();
    }
}
````

## UserManager

During the integration of Elsa, we faced an error of Serialization and Deserialization of ```IdentityOptions``` in UserManager's ```InitializeOptionsAsync``` method. To overcome this problem, you should override the related method in UserManager.cs as shown below;

````c#
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
````

This approach manually creates a ```IdentityOptions``` and manually sets its fields one by one. The previous version uses AutoMapper which causes a problem.

## Navigation

Finally, let's add Elsa Dashboard as a menu item to our app. In order to do that, first create a new localization in the localization xml file as shown below;

````xml
<text name="Workflows">Workflows</text>
````

After that, define a constant in **AppPageNames.cs** as shown below right under ```DynamicEntityProperties``` definition;

````c#
public const string Elsa = "Elsa.Main";
````

Let's also define a permission for our new page. To do that, create a const in **AppPermisisons.cs** as shown below right under ```Pages_Administration_DynamicEntityPropertyValue_Delete``` ;

````c#
public const string Pages_Workflows = "Pages.Workflows";
````

and define a permission in **AppAuthorizationProvider.cs** as show below right after definition of ```AppPermissions.Pages_DemoUiComponents``` ;

````c#
pages.CreateChildPermission(AppPermissions.Pages_Workflows, L("Workflows"));
````

after all, add a menu item to **AppNavigationProvider.cs** as shown below;

````c#
.AddItem(new MenuItemDefinition(
    AppPageNames.Common.Elsa,
    L("Workflows"),
    url: "/Workflows",
    icon: "flaticon-map",
    permissionDependency: new SimplePermissionDependency(AppPermissions.Pages_Workflows)
)
````

That's all. You can now run the application and navigate to ```/Workflows``` page and see Elsa Dashboard.

## Sample Workflow

You can create a sample workflow as explained in [Elsa Documentation](https://elsa-workflows.github.io/elsa-core/docs/next/quickstarts/quickstarts-aspnetcore-server-dashboard-and-api-endpoints#the-workflow). This sample workflow starts with a request to ```/hello-world``` endpoint and returns the specified HTTP response.
