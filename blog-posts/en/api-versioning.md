# ASP.NET Core API Versioning in ASP.NET Zero

APIs are the backbone of modern applications. As your API evolves, adding new features and updating old ones becomes inevitable. This is where API versioning comes into play. But why is API versioning important, and how do we implement it in ASP.NET Core?

## Why API Versioning?

Many clients maybe using your API, each one maybe using different versions. If you update your API without versioning, older clients may break. API versioning allows you to introduce new features while keeping the older versions functional for existing clients.

There are several reasons you might need to make changes:

- **Modifications to the request structure:** This could involve adding, removing, or adjusting parameters sent by the client.
- **Adjustments to the response format:** Similar changes could occur in the data returned by the API, potentially altering the structure or content.
- **Behavioral changes in existing operations:** Sometimes, the functionality of a specific action might shift, leading to subtle differences in how it performs.
- **Modifying accepted HTTP methods:** Though rare, there might be situations where you adjust which HTTP methods (GET, POST, etc.) an action can handle.
- **Changing HTTP status codes:** The API might return different status codes to reflect new error handling or success scenarios more accurately.

## How to Implement API Versioning in ASP.NET Zero

There is a strong library for API versioning in ASP.NET Core called `Asp.Versioning.Mvc.ApiExplorer`.

### Step 1: Install the NuGet Package

```bash
dotnet add package Asp.Versioning.Mvc.ApiExplorer
```

### Step 2: Configuration

In ASP.NET Zero we need to add some classes to configure API versioning.

* Create a new folder called `ApiVersioning` in the `Web.Core` project. And add the following classes:

*AbpAppServiceApiVersionSpecification.cs*
```csharp
using System.Collections.Generic;
using Abp.Application.Services;
using Asp.Versioning.ApplicationModels;
using Microsoft.AspNetCore.Mvc.ApplicationModels;

namespace ApiVersionSampleDemo.Web.Startup.ApiVersioning;

public class AbpAppServiceApiVersionSpecification : IApiControllerSpecification
{
    public bool IsSatisfiedBy(ControllerModel controller)
    {
        var typeInfo = controller.ControllerType;
        var type = controller.ControllerType.AsType(); ;
        if (!typeof(IApplicationService).IsAssignableFrom(type) ||
            !typeInfo.IsPublic || typeInfo.IsAbstract || typeInfo.IsGenericType)
        {
            return false;
        }
        if (!(controller.Attributes is List<object> attributes))
        {
            return false;
        }
        
        return true;
    }
}
```

*MyAbpAppServiceConvention*
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using Abp;
using Abp.Application.Services;
using Abp.AspNetCore.Configuration;
using Abp.Collections.Extensions;
using Abp.Extensions;
using Abp.Web.Api.ProxyScripting.Generators;
using Asp.Versioning;
using Castle.Windsor.MsDependencyInjection;
using JetBrains.Annotations;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.ActionConstraints;
using Microsoft.AspNetCore.Mvc.ApplicationModels;
using Microsoft.AspNetCore.Mvc.ModelBinding;
using Microsoft.Extensions.DependencyInjection;

namespace ApiVersionSampleDemo.Web.Startup.ApiVersioning;

public class MyAbpAppServiceConvention : IApplicationModelConvention
{
    private readonly Lazy<AbpAspNetCoreConfiguration> _configuration;

    public MyAbpAppServiceConvention(IServiceCollection services)
    {
        _configuration = new Lazy<AbpAspNetCoreConfiguration>(() =>
        {
            return services
                .GetSingletonService<AbpBootstrapper>()
                .IocManager
                .Resolve<AbpAspNetCoreConfiguration>();
        }, true);
    }

    public void Apply(ApplicationModel application)
    {
        foreach (var controller in application.Controllers)
        {
            if (controller.ControllerName.Contains("Order"))
            {
                var x = 1;
            }
            
            var type = controller.ControllerType.AsType();
            var configuration = GetControllerSettingOrNull(type);

            if (typeof(IApplicationService).GetTypeInfo().IsAssignableFrom(type))
            {
                controller.ControllerName = controller.ControllerName.RemovePostFix(ApplicationService.CommonPostfixes);
                configuration?.ControllerModelConfigurer(controller);

                ConfigureCacheControl(controller, _configuration.Value.DefaultResponseCacheAttributeForAppServices);
                ConfigureArea(controller, configuration);
                ConfigureRemoteService(controller, configuration);
            }
            else
            {
                var remoteServiceAtt =
                    MyReflectionHelper.GetSingleAttributeOrDefault<RemoteServiceAttribute>(type.GetTypeInfo());
                if (remoteServiceAtt != null && remoteServiceAtt.IsEnabledFor(type))
                {
                    ConfigureCacheControl(controller, _configuration.Value.DefaultResponseCacheAttributeForControllers);
                    ConfigureRemoteService(controller, configuration);
                }
            }
        }
    }

    private void ConfigureCacheControl(ControllerModel controller, ResponseCacheAttribute responseCacheAttribute)
    {
        if (responseCacheAttribute == null)
        {
            return;
        }

        if (controller.Filters.Any(filter => typeof(ResponseCacheAttribute).IsAssignableFrom(filter.GetType())))
        {
            return;
        }

        controller.Filters.Add(responseCacheAttribute);
    }

    private void ConfigureArea(ControllerModel controller, [CanBeNull] AbpControllerAssemblySetting configuration)
    {
        if (configuration == null)
        {
            return;
        }

        if (controller.RouteValues.ContainsKey("area"))
        {
            return;
        }

        controller.RouteValues["area"] = configuration.ModuleName;
    }

    private void ConfigureRemoteService(ControllerModel controller,
        [CanBeNull] AbpControllerAssemblySetting configuration)
    {
        ConfigureApiExplorer(controller);
        ConfigureSelector(controller, configuration);
        ConfigureParameters(controller);
    }

    private void ConfigureParameters(ControllerModel controller)
    {
        foreach (var action in controller.Actions)
        {
            foreach (var prm in action.Parameters)
            {
                if (prm.BindingInfo != null)
                {
                    continue;
                }

                if (!MyTypeHelper.IsPrimitiveExtendedIncludingNullable(prm.ParameterInfo.ParameterType))
                {
                    if (CanUseFormBodyBinding(action, prm))
                    {
                        prm.BindingInfo = BindingInfo.GetBindingInfo(new[] {new FromBodyAttribute()});
                    }
                }
            }
        }
    }

    private bool CanUseFormBodyBinding(ActionModel action, ParameterModel parameter)
    {
        if (_configuration.Value.FormBodyBindingIgnoredTypes.Any(t =>
                t.IsAssignableFrom(parameter.ParameterInfo.ParameterType)))
        {
            return false;
        }

        foreach (var selector in action.Selectors)
        {
            if (selector.ActionConstraints == null)
            {
                continue;
            }

            foreach (var actionConstraint in selector.ActionConstraints)
            {
                var httpMethodActionConstraint = actionConstraint as HttpMethodActionConstraint;
                if (httpMethodActionConstraint == null)
                {
                    continue;
                }

                if (httpMethodActionConstraint.HttpMethods.All(hm => hm.IsIn("GET", "DELETE", "TRACE", "HEAD")))
                {
                    return false;
                }
            }
        }

        return true;
    }

    private void ConfigureApiExplorer(ControllerModel controller)
    {
        if (controller.ApiExplorer.GroupName.IsNullOrEmpty())
        {
            controller.ApiExplorer.GroupName = controller.ControllerName;
        }

        if (controller.ApiExplorer.IsVisible == null)
        {
            var controllerType = controller.ControllerType.AsType();
            var remoteServiceAtt =
                MyReflectionHelper.GetSingleAttributeOrDefault<RemoteServiceAttribute>(controllerType.GetTypeInfo());
            if (remoteServiceAtt != null)
            {
                controller.ApiExplorer.IsVisible =
                    remoteServiceAtt.IsEnabledFor(controllerType) &&
                    remoteServiceAtt.IsMetadataEnabledFor(controllerType);
            }
            else
            {
                controller.ApiExplorer.IsVisible = true;
            }
        }

        foreach (var action in controller.Actions)
        {
            ConfigureApiExplorer(action);
        }
    }

    private void ConfigureApiExplorer(ActionModel action)
    {
        if (action.ApiExplorer.IsVisible == null)
        {
            var remoteServiceAtt =
                MyReflectionHelper.GetSingleAttributeOrDefault<RemoteServiceAttribute>(action.ActionMethod);
            if (remoteServiceAtt != null)
            {
                action.ApiExplorer.IsVisible =
                    remoteServiceAtt.IsEnabledFor(action.ActionMethod) &&
                    remoteServiceAtt.IsMetadataEnabledFor(action.ActionMethod);
            }
        }
    }

    private void ConfigureSelector(ControllerModel controller, [CanBeNull] AbpControllerAssemblySetting configuration)
    {
        RemoveEmptySelectors(controller.Selectors);

        if (controller.Selectors.Any(selector => selector.AttributeRouteModel != null))
        {
            return;
        }

        var moduleName = GetModuleNameOrDefault(controller.ControllerType.AsType());

        foreach (var action in controller.Actions)
        {
            ConfigureSelector(moduleName, controller.ControllerName, action, configuration);
        }
    }

    private void ConfigureSelector(string moduleName, string controllerName, ActionModel action,
        [CanBeNull] AbpControllerAssemblySetting configuration)
    {
        RemoveEmptySelectors(action.Selectors);

        var remoteServiceAtt =
            MyReflectionHelper.GetSingleAttributeOrDefault<RemoteServiceAttribute>(action.ActionMethod);
        if (remoteServiceAtt != null && !remoteServiceAtt.IsEnabledFor(action.ActionMethod))
        {
            return;
        }

        if (!action.Selectors.Any())
        {
            AddAbpServiceSelector(moduleName, controllerName, action, configuration);
        }
        else
        {
            NormalizeSelectorRoutes(moduleName, controllerName, action, configuration);
        }
    }

    private void AddAbpServiceSelector(string moduleName, string controllerName, ActionModel action,
        [CanBeNull] AbpControllerAssemblySetting configuration)
    {
        var abpServiceSelectorModel = new SelectorModel
        {
            AttributeRouteModel = CreateAbpServiceAttributeRouteModel(moduleName, controllerName, action)
        };

        var httpMethod = SelectHttpMethod(action, configuration);

        abpServiceSelectorModel.ActionConstraints.Add(new HttpMethodActionConstraint(new[] {httpMethod}));

        action.Selectors.Add(abpServiceSelectorModel);
    }

    private string SelectHttpMethod(ActionModel action, AbpControllerAssemblySetting configuration)
    {
        return configuration?.UseConventionalHttpVerbs == true
            ? ProxyScriptingHelper.GetConventionalVerbForMethodName(action.ActionName)
            : ProxyScriptingHelper.DefaultHttpVerb;
    }

    private void NormalizeSelectorRoutes(string moduleName, string controllerName, ActionModel action,
        [CanBeNull] AbpControllerAssemblySetting configuration)
    {
        foreach (var selector in action.Selectors)
        {
            if (!selector.ActionConstraints.OfType<HttpMethodActionConstraint>().Any())
            {
                var httpMethod = SelectHttpMethod(action, configuration);
                selector.ActionConstraints.Add(new HttpMethodActionConstraint(new[] {httpMethod}));
            }

            if (selector.AttributeRouteModel == null)
            {
                selector.AttributeRouteModel = CreateAbpServiceAttributeRouteModel(
                    moduleName,
                    controllerName,
                    action
                );
            }
        }
    }

    private string GetModuleNameOrDefault(Type controllerType)
    {
        return GetControllerSettingOrNull(controllerType)?.ModuleName ??
               AbpControllerAssemblySetting.DefaultServiceModuleName;
    }

    [CanBeNull]
    private AbpControllerAssemblySetting GetControllerSettingOrNull(Type controllerType)
    {
        var settings = _configuration.Value.ControllerAssemblySettings.GetSettings(controllerType);
        return settings.FirstOrDefault(setting => setting.TypePredicate(controllerType));
    }

    private static AttributeRouteModel CreateAbpServiceAttributeRouteModel(string moduleName, string controllerName,
        ActionModel action)
    {
        List<ApiVersion> controllerVersions = new List<ApiVersion>();
        if (action.Attributes.Any(x => x is ApiVersionAttribute))
        {
            controllerVersions = action.Attributes.OfType<ApiVersionAttribute>().SelectMany(x => x.Versions).ToList();
        }
        else if (action.Controller.Attributes.Any(x => x is ApiVersionAttribute))
        {
            controllerVersions = action.Controller.ControllerType.GetCustomAttributes(true)
                .OfType<ApiVersionAttribute>().SelectMany(x => x.Versions).ToList();
        }

        if (!controllerVersions.Any())
        {
            controllerVersions.Add(new ApiVersion(1, 0));
        }

        var version = controllerVersions.First().ToString();

        return new AttributeRouteModel(
            new RouteAttribute(
                $"api/v{version}/services/{moduleName}/{controllerName}/{action.ActionName}"
            )
        );
    }

    private static void RemoveEmptySelectors(IList<SelectorModel> selectors)
    {
        selectors
            .Where(IsEmptySelector)
            .ToList()
            .ForEach(s => selectors.Remove(s));
    }

    private static bool IsEmptySelector(SelectorModel selector)
    {
        return selector.AttributeRouteModel == null
               && selector.ActionConstraints.IsNullOrEmpty()
               && selector.EndpointMetadata.IsNullOrEmpty();
    }
}
```

*MyReflectionHelper*
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using System.Threading.Tasks;

namespace ApiVersionSampleDemo.Web.Startup.ApiVersioning;

internal static class MyReflectionHelper
{
    /// <summary>
    /// Checks whether <paramref name="givenType"/> implements/inherits <paramref name="genericType"/>.
    /// </summary>
    /// <param name="givenType">Type to check</param>
    /// <param name="genericType">Generic type</param>
    public static bool IsAssignableToGenericType(Type givenType, Type genericType)
    {
        var givenTypeInfo = givenType.GetTypeInfo();

        if (givenTypeInfo.IsGenericType && givenType.GetGenericTypeDefinition() == genericType)
        {
            return true;
        }

        foreach (var interfaceType in givenType.GetInterfaces())
        {
            if (interfaceType.GetTypeInfo().IsGenericType && interfaceType.GetGenericTypeDefinition() == genericType)
            {
                return true;
            }
        }

        if (givenTypeInfo.BaseType == null)
        {
            return false;
        }

        return IsAssignableToGenericType(givenTypeInfo.BaseType, genericType);
    }

    /// <summary>
    /// Gets a list of attributes defined for a class member and it's declaring type including inherited attributes.
    /// </summary>
    /// <param name="inherit">Inherit attribute from base classes</param>
    /// <param name="memberInfo">MemberInfo</param>
    public static List<object> GetAttributesOfMemberAndDeclaringType(MemberInfo memberInfo, bool inherit = true)
    {
        var attributeList = new List<object>();

        attributeList.AddRange(memberInfo.GetCustomAttributes(inherit));

        if (memberInfo.DeclaringType != null)
        {
            attributeList.AddRange(memberInfo.DeclaringType.GetTypeInfo().GetCustomAttributes(inherit));
        }

        return attributeList;
    }

    /// <summary>
    /// Gets a list of attributes defined for a class member and type including inherited attributes.
    /// </summary>
    /// <param name="memberInfo">MemberInfo</param>
    /// <param name="type">Type</param>
    /// <param name="inherit">Inherit attribute from base classes</param>
    public static List<object> GetAttributesOfMemberAndType(MemberInfo memberInfo, Type type, bool inherit = true)
    {
        var attributeList = new List<object>();
        attributeList.AddRange(memberInfo.GetCustomAttributes(inherit));
        attributeList.AddRange(type.GetTypeInfo().GetCustomAttributes(inherit));
        return attributeList;
    }

    /// <summary>
    /// Gets a list of attributes defined for a class member and it's declaring type including inherited attributes.
    /// </summary>
    /// <typeparam name="TAttribute">Type of the attribute</typeparam>
    /// <param name="memberInfo">MemberInfo</param>
    /// <param name="inherit">Inherit attribute from base classes</param>
    public static List<TAttribute> GetAttributesOfMemberAndDeclaringType<TAttribute>(MemberInfo memberInfo,
        bool inherit = true)
        where TAttribute : Attribute
    {
        var attributeList = new List<TAttribute>();

        if (memberInfo.IsDefined(typeof(TAttribute), inherit))
        {
            attributeList.AddRange(memberInfo.GetCustomAttributes(typeof(TAttribute), inherit).Cast<TAttribute>());
        }

        if (memberInfo.DeclaringType != null &&
            memberInfo.DeclaringType.GetTypeInfo().IsDefined(typeof(TAttribute), inherit))
        {
            attributeList.AddRange(memberInfo.DeclaringType.GetTypeInfo()
                .GetCustomAttributes(typeof(TAttribute), inherit).Cast<TAttribute>());
        }

        return attributeList;
    }

    /// <summary>
    /// Gets a list of attributes defined for a class member and type including inherited attributes.
    /// </summary>
    /// <typeparam name="TAttribute">Type of the attribute</typeparam>
    /// <param name="memberInfo">MemberInfo</param>
    /// <param name="type">Type</param>
    /// <param name="inherit">Inherit attribute from base classes</param>
    public static List<TAttribute> GetAttributesOfMemberAndType<TAttribute>(MemberInfo memberInfo, Type type,
        bool inherit = true)
        where TAttribute : Attribute
    {
        var attributeList = new List<TAttribute>();

        if (memberInfo.IsDefined(typeof(TAttribute), inherit))
        {
            attributeList.AddRange(memberInfo.GetCustomAttributes(typeof(TAttribute), inherit).Cast<TAttribute>());
        }

        if (type.GetTypeInfo().IsDefined(typeof(TAttribute), inherit))
        {
            attributeList.AddRange(type.GetTypeInfo().GetCustomAttributes(typeof(TAttribute), inherit)
                .Cast<TAttribute>());
        }

        return attributeList;
    }

    /// <summary>
    /// Tries to gets an of attribute defined for a class member and it's declaring type including inherited attributes.
    /// Returns default value if it's not declared at all.
    /// </summary>
    /// <typeparam name="TAttribute">Type of the attribute</typeparam>
    /// <param name="memberInfo">MemberInfo</param>
    /// <param name="defaultValue">Default value (null as default)</param>
    /// <param name="inherit">Inherit attribute from base classes</param>
    public static TAttribute GetSingleAttributeOfMemberOrDeclaringTypeOrDefault<TAttribute>(MemberInfo memberInfo,
        TAttribute defaultValue = default(TAttribute), bool inherit = true)
        where TAttribute : class
    {
        return memberInfo.GetCustomAttributes(true).OfType<TAttribute>().FirstOrDefault()
               ?? memberInfo.ReflectedType?.GetTypeInfo().GetCustomAttributes(true).OfType<TAttribute>()
                   .FirstOrDefault()
               ?? defaultValue;
    }

    /// <summary>
    /// Tries to gets an of attribute defined for a class member and it's declaring type including inherited attributes.
    /// Returns default value if it's not declared at all.
    /// </summary>
    /// <typeparam name="TAttribute">Type of the attribute</typeparam>
    /// <param name="memberInfo">MemberInfo</param>
    /// <param name="defaultValue">Default value (null as default)</param>
    /// <param name="inherit">Inherit attribute from base classes</param>
    public static TAttribute GetSingleAttributeOrDefault<TAttribute>(MemberInfo memberInfo,
        TAttribute defaultValue = default(TAttribute), bool inherit = true)
        where TAttribute : Attribute
    {
        //Get attribute on the member
        if (memberInfo.IsDefined(typeof(TAttribute), inherit))
        {
            return memberInfo.GetCustomAttributes(typeof(TAttribute), inherit).Cast<TAttribute>().First();
        }

        return defaultValue;
    }

    /// <summary>
    /// Gets a property by it's full path from given object
    /// </summary>
    /// <param name="obj">Object to get value from</param>
    /// <param name="objectType">Type of given object</param>
    /// <param name="propertyPath">Full path of property</param>
    /// <returns></returns>
    internal static object GetPropertyByPath(object obj, Type objectType, string propertyPath)
    {
        var property = obj;
        var currentType = objectType;
        var objectPath = currentType.FullName;
        var absolutePropertyPath = propertyPath;
        if (absolutePropertyPath.StartsWith(objectPath))
        {
            absolutePropertyPath = absolutePropertyPath.Replace(objectPath + ".", "");
        }

        foreach (var propertyName in absolutePropertyPath.Split('.'))
        {
            property = currentType.GetProperty(propertyName);
            currentType = ((PropertyInfo) property).PropertyType;
        }

        return property;
    }

    /// <summary>
    /// Gets value of a property by it's full path from given object
    /// </summary>
    /// <param name="obj">Object to get value from</param>
    /// <param name="objectType">Type of given object</param>
    /// <param name="propertyPath">Full path of property</param>
    /// <returns></returns>
    internal static object GetValueByPath(object obj, Type objectType, string propertyPath)
    {
        var value = obj;
        var currentType = objectType;
        var objectPath = currentType.FullName;
        var absolutePropertyPath = propertyPath;
        if (absolutePropertyPath.StartsWith(objectPath))
        {
            absolutePropertyPath = absolutePropertyPath.Replace(objectPath + ".", "");
        }

        foreach (var propertyName in absolutePropertyPath.Split('.'))
        {
            var property = currentType.GetProperty(propertyName);
            value = property.GetValue(value, null);
            currentType = property.PropertyType;
        }

        return value;
    }

    /// <summary>
    /// Sets value of a property by it's full path on given object
    /// </summary>
    /// <param name="obj"></param>
    /// <param name="objectType"></param>
    /// <param name="propertyPath"></param>
    /// <param name="value"></param>
    internal static void SetValueByPath(object obj, Type objectType, string propertyPath, object value)
    {
        var currentType = objectType;
        PropertyInfo property;
        var objectPath = currentType.FullName;
        var absolutePropertyPath = propertyPath;
        if (absolutePropertyPath.StartsWith(objectPath))
        {
            absolutePropertyPath = absolutePropertyPath.Replace(objectPath + ".", "");
        }

        var properties = absolutePropertyPath.Split('.');

        if (properties.Length == 1)
        {
            property = objectType.GetProperty(properties.First());
            property.SetValue(obj, value);
            return;
        }

        for (int i = 0; i < properties.Length - 1; i++)
        {
            property = currentType.GetProperty(properties[i]);
            obj = property.GetValue(obj, null);
            currentType = property.PropertyType;
        }

        property = currentType.GetProperty(properties.Last());
        property.SetValue(obj, value);
    }

    internal static bool IsPropertyGetterSetterMethod(MethodInfo method, Type type)
    {
        if (!method.IsSpecialName)
        {
            return false;
        }

        if (method.Name.Length < 5)
        {
            return false;
        }

        return type.GetProperty(method.Name.Substring(4),
            BindingFlags.Instance | BindingFlags.Static | BindingFlags.NonPublic) != null;
    }

    internal static async Task<object> InvokeAsync(MethodInfo method, object obj, params object[] parameters)
    {
        var task = (Task) method.Invoke(obj, parameters);
        await task;
        var resultProperty = task.GetType().GetProperty("Result");
        return resultProperty?.GetValue(task);
    }
}
```

*MyTypeHelper*
```csharp
using System;
using System.Reflection;

namespace ApiVersionSampleDemo.Web.Startup.ApiVersioning;

internal static class MyTypeHelper
{
    public static bool IsFunc(object obj)
    {
        if (obj == null)
        {
            return false;
        }

        var type = obj.GetType();
        if (!type.GetTypeInfo().IsGenericType)
        {
            return false;
        }

        return type.GetGenericTypeDefinition() == typeof(Func<>);
    }

    public static bool IsFunc<TReturn>(object obj)
    {
        return obj != null && obj.GetType() == typeof(Func<TReturn>);
    }

    public static bool IsPrimitiveExtendedIncludingNullable(Type type, bool includeEnums = false)
    {
        if (IsPrimitiveExtended(type, includeEnums))
        {
            return true;
        }

        if (type.GetTypeInfo().IsGenericType && type.GetGenericTypeDefinition() == typeof(Nullable<>))
        {
            return IsPrimitiveExtended(type.GenericTypeArguments[0], includeEnums);
        }

        return false;
    }

    private static bool IsPrimitiveExtended(Type type, bool includeEnums)
    {
        if (type.GetTypeInfo().IsPrimitive)
        {
            return true;
        }

        if (includeEnums && type.GetTypeInfo().IsEnum)
        {
            return true;
        }

        return type == typeof(string) ||
               type == typeof(decimal) ||
               type == typeof(DateTime) ||
               type == typeof(DateTimeOffset) ||
               type == typeof(TimeSpan) ||
               type == typeof(Guid);
    }
}
```

* Then in Startup.cs file, add the following code to the `ConfigureServices` method:

```csharp
services.PostConfigure<MvcOptions>(opt =>
{
    opt.Conventions.RemoveAt(1);
    opt.Conventions.Add(new MyAbpAppServiceConvention(services));
});
```

### Step 3: Configure API Versioning

In the `Startup.cs` file, add the following code to the `ConfigureServices` method:

```csharp
services
    .AddApiVersioning(options =>
    {
        options.DefaultApiVersion = new ApiVersion(1, 0);
        options.ReportApiVersions = true;
        options.AssumeDefaultVersionWhenUnspecified = true;
        options.ApiVersionReader = ApiVersionReader.Combine(
            new UrlSegmentApiVersionReader(),
            new HeaderApiVersionReader("X-Api-Version"));
    })
    .AddApiExplorer(options =>
    {
        options.GroupNameFormat = "'v'VVV";
        options.SubstituteApiVersionInUrl = true;
    });
```

### Step 4: Add API Versioning to the Controller

To add versioning to a controller, use the `ApiVersion` attribute. Here is an example:

* Create a new folder named V1 in the Controllers folder. Then add a new controller named `MathController.cs` with the following code:

```csharp
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/[controller]/[action]")]
[ApiController]
public class MathController : ControllerBase
{
    // Controller actions
    
    [HttpGet]
    public IActionResult Add(int a, int b)
    {
        return Ok(a + b);
    }
}
```

* Create a new folder named V2 in the Controllers folder. Then add a new controller named `MathController.cs` with the following code:

```csharp
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/[controller]/[action]")]
[ApiController]
public class MathController : ControllerBase
{
    // Controller actions
    
    [HttpGet]
    public IActionResult Add(int a, int b, int c)
    {
        return Ok(a + b + c);
    }
}
```

### Step 5: Using API Versioning in AppServices

If you are a ASP.NET Zero developer, you can use the following code to add API versioning in your AppServices:

Create folders named `V1` and `V2` in the `Application` folder. Then add a new AppService named `MathAppService.cs` with the following code:

*V1/MathAppService.cs*
```csharp
using Abp.Application.Services;
using Asp.Versioning;
using Microsoft.AspNetCore.Mvc;

namespace ApiVersionSampleDemo.Math.V1;

public interface IMathAppService
{
    int Sum(int number1, int number2);
}

[ApiVersion("1.0")]
[ApiController]
[Route("api/v{version:apiVersion}/[controller]/[action]")]
public class MathAppService : ApplicationService, IMathAppService
{
    [HttpGet]
    public int Sum(int number1, int number2)
    {
        return number1 + number2;
    }
}
```

*V2/MathAppService.cs*
```csharp
using Abp.Application.Services;
using Asp.Versioning;
using Microsoft.AspNetCore.Mvc;

namespace ApiVersionSampleDemo.Math.V2;

public interface IMathAppService
{
    int Sum(int number1, int number2, int number3);
}

[ApiVersion("2.0")]
[ApiController]
[Route("api/v{version:apiVersion}/[controller]/[action]")]
public class MathAppService : ApplicationService, IMathAppService
{
    [HttpGet]
    public int Sum(int number1, int number2, int number3)
    {
        return number1 + number2 + number3;
    }
}
```

### Step 6: Test the API

Now you can test the API using different versions. For example, you can use the following URLs:

![Api Versioning Swagger](/Images/Blog/api-versioning-swagger.png)

- `https://localhost:44301/api/v1/math/sum?number1=1&number2=2`
- `https://localhost:44301/api/v2/math/sum?number1=1&number2=2&number3=3`

You can see supported API version in response header `api-supported-versions`.

![Api Versioning Response Header](/Images/Blog/api-versioning-supported-versions.png)

## Conclusion

In this blog post, we learned how to implement API versioning in ASP.NET Core using the `Asp.Versioning.Mvc.ApiExplorer` library. We also implemented API versioning for ASP.NET ZERO. By following these steps, you can easily version your APIs and keep them up-to-date with new features.

The ASP.NET ZERO sample is here https://github.com/aspnetzero/aspnet-zero-samples/tree/master/ApiVersionSampleDemo