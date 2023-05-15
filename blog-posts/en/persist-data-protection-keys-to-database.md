# Persist Data Protection Keys to Database in ASP.NET Zero

ASP.NET Core provides a very sophisticated approach for protecting data. You can check [ASP.NET Core Data Protection Documentation](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/introduction) for more information.

Just like you do in any other ASP.NET Core app, if you want to use **data protection** in ASP.NET Zero, you can easily configure it. However, if you want to store data protection keys in **database** (see [documentation](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview)), you will face an error. That's because ASP.NET Core will try to create configured DbContext but at that time DbContext will not be configured by ASP.NET Zero. To overcome this problem, we need to configure data protection **a bit different**.

## Default Configuration

First of all, you need to add related NuGet package to your project and add `DataProtectionKey` entity to your DbContext as explained [here](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/configuration/overview). We will just configure data projecttion in a different way.

## Dependency Injection

By default, ASP.NET Core's data protection must be configured in `Startup.cs` file. However, in our case, we must configure it in ASP.NET Zero's EF Core module. To do that, we must access the `IServiceCollection` in our EF Core module.

Create classes below in the project;

```csharp
public interface IServiceCollectionProvider 
{
    IServiceCollection ServiceCollection { get; }
}

public sealed class ServiceCollectionProvider: IServiceCollectionProvider
{
    public ServiceCollectionProvider(IServiceCollection serviceCollection)
    {
        ServiceCollection = serviceCollection;
    }

    public IServiceCollection ServiceCollection { get; }
}
```

Then, we can register IServiceCollectionProvider in our Startup.cs file as shown below;

```csharp
services.AddSingleton<IServiceCollectionProvider>(new ServiceCollectionProvider(services));
```

## Configure Data Protection

Finally, configure data protection in the `PostInitialize` method of the EFCore module as shown below;

```csharp
var serviceCollectionProvider = IocManager.IocContainer.Resolve<IServiceCollectionProvider>();
serviceCollectionProvider.ServiceCollection.AddDataProtection().PersistKeysToDbContext<YourDbContext>()
	.SetApplicationName("YourApplicationName");
```   

