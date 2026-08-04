**Title:** Using AutoMapper After Migrating to Mapperly
**Description:** ASP.NET Zero switched its default object-to-object mapper from AutoMapper to Mapperly in v15.2. This is a complete, step-by-step guide to reverting your solution back to AutoMapper: choosing an AutoMapper version under the new commercial license, building and publishing Abp.AutoMapper to a local NuGet feed, swapping the ABP module, restoring CustomDtoMapper and the [AutoMap*] attributes, converting the generated Mapperly mappers, and validating the result.

# Using AutoMapper After Migrating to Mapperly

Starting with **ASP.NET Zero v15.2**, the default object to object mapping solution is [Mapperly](https://mapperly.riok.app/) instead of [AutoMapper](https://automapper.io/). The template ships compile-time generated mappers, and the ABP module registered for `IObjectMapper` is `AbpMapperlyModule` rather than `AbpAutoMapperModule`.

For new projects that is the right default. But many long-lived ASP.NET Zero solutions have years of AutoMapper investment behind them, a `CustomDtoMapper.cs` with several hundred `CreateMap` calls, `[AutoMap]`, `[AutoMapFrom]` and `[AutoMapTo]` attributes across DTOs and view models, custom `ITypeConverter` and `IValueResolver` implementations, and `ProjectTo` queries that push projections into SQL. Rewriting all of that during an upgrade window is not always the right call, and for some teams AutoMapper is simply the better fit.

This post is a complete runbook for **reverting an upgraded ASP.NET Zero solution back to AutoMapper**. Every step is spelled out, with the exact files involved.

> Written against ASP.NET Zero v15.2 – v15.4 and ABP 11.x. Paths use the default `MyCompanyName.AbpZeroTemplate` naming; substitute your own company and project names.

## What You Are Undoing

The migration commit in the ASP.NET Zero template reaches into every layer of the solution. A full revert has to reverse five distinct things:

1. **The ABP module**, `AbpAutoMapperModule` → `AbpMapperlyModule` in `AbpZeroTemplateCoreModule` and `AbpZeroTemplateMauiModule`.
2. **`CustomDtoMapper.cs`**, deleted from both the `.Application` project and the `.GraphQL` project, along with the `Configurators.Add(...)` registrations in the corresponding modules.
3. **`[AutoMap*]` attributes**, removed from 28 files across five projects: cache items, GraphQL DTOs, MAUI persistence models, and MVC view models.
4. **Generated mapper classes**, added across six projects.
5. **Package references and build settings**, `Abp.AutoMapper` → `Abp.Mapperly` in four `.csproj` files, plus the `RMG020;RMG012;RMG066` `NoWarn` suppressions added to `aspnet-core/common.props`.

Here is the mapper inventory you will be converting or deleting:

| Project | Mapper files | `[Mapper]` classes |
|---|---:|---:|
| `.Application` | 19 | 80 |
| `.Web.Mvc` | 9 | 13 |
| `.Maui` | 4 | 14 |
| `.GraphQL` | 3 | 5 |
| `.Core` | 1 | 1 |
| `.Web.Core` | 1 | 1 |
| **Total** | **37** | **114** |

For a solution that has not diverged much from the template, this is roughly a day of focused work plus a testing pass. Nothing changes on the frontend: Angular, React and MVC client code is untouched.

**Before you start, create a branch and make sure your pre-upgrade commit is reachable.** Most of the work below is recovering files from your own git history rather than writing them.

## Prerequisite 1: Choose Your AutoMapper Version

This decision determines everything downstream, so make it first.

AutoMapper moved under Lucky Penny Software and [went commercial](https://github.com/LuckyPennySoftware/AutoMapper/discussions/4536) with v15.0:

| Version line | License | Notes |
|---|---|---|
| **14.0.0** and earlier | MIT | Last permissively licensed release. `net8.0` target only, which a `net10.0` project consumes fine. |
| **15.0+** (currently 16.x) | Dual: [RPL-1.5](https://github.com/LuckyPennySoftware/AutoMapper/blob/main/LICENSE.md) or commercial | Free tier for organizations under **$5M USD gross annual revenue**, non-profits under $5M budget, and educational/non-production use. A license key is expected for auditing. Targets `net8.0`, `net9.0`, `net10.0`, `net471`, `netstandard2.0`. |

`AutoMapper.Collection`, which `Abp.AutoMapper` needs at runtime, pins you to one line or the other:

| AutoMapper.Collection | Requires AutoMapper |
|---|---|
| 11.0.0 | `[14.0.0, 15.0.0)` |
| 12.0.0 | `[15.0.1, 16.0.0)` |
| 13.0.0 | `[15.0.1, 17.0.0)` |

So there are two viable combinations:

- **Free path:** AutoMapper 14.0.0 + AutoMapper.Collection 11.0.0. This is exactly what `Abp.AutoMapper.csproj` already references, so the package builds unmodified. Choose this unless you have a specific reason not to.
- **Licensed path:** AutoMapper 15.x/16.x + AutoMapper.Collection 12.0.0/13.0.0. Requires patching `AbpAutoMapperModule` (covered in Prerequisite 2, step 4) because AutoMapper 15 removed the constructor ABP uses.

> `AutoMapper.Collection` is not optional if you use `[AutoMapKey]`. `AutoMapperConfigurationExtensions.CreateAutoAttributeMaps` loads the assembly **by name** at runtime to wire up `EqualityComparison`, so dropping the package turns that into a runtime failure, not a compile error.

## Prerequisite 2: Get the `Abp.AutoMapper` Package

`Abp.AutoMapper` still lives in the ABP source tree and is still listed in the packaging script next to `Abp.Mapperly`. It is formally deprecated:

```csharp
[Obsolete("Abp.AutoMapper package is deprecated. Use Abp.Mapperly package instead.")]
[DependsOn(typeof(AbpKernelModule))]
public class AbpAutoMapperModule : AbpModule
```

Deprecated means no new features and no guarantee it survives a future major; it does not mean removed. It still targets `net10.0` and still works.

### Step 1: Check your ASP.NET Zero feed first

ABP 11.x packages ship through the ASP.NET Zero NuGet feed configured in `aspnet-core/NuGet.Config`, not nuget.org (the public `Abp.*` packages stop at the 10.5.0 line). Check whether the version you need is already published:

```powershell
dotnet package search Abp.AutoMapper `
    --source https://nuget.aspnetzero.com/<YOUR-API-KEY>/v3/index.json `
    --exact-match
```

If a matching 11.x version comes back and you are on the free AutoMapper path, you are done with this prerequisite; skip to [Step 1: Swap the Package References](#step-1-swap-the-package-references).

Otherwise, build it yourself.

### Step 2: Build `Abp.AutoMapper` from source

Clone the ABP repository and check out the tag matching your ABP version:

```powershell
git clone https://github.com/aspnetzero/aspnetboilerplate.git
cd aspnetboilerplate
git checkout v11.3
```

Confirm `common.props` at the repository root carries the version you expect:

```xml
<PropertyGroup>
    <Version>11.3.0</Version>
    ...
</PropertyGroup>
```

Pack just that one project; there is no need to build the whole solution:

```powershell
cd src/Abp.AutoMapper
dotnet pack -c Release -o D:\LocalNuget
```

This produces `D:\LocalNuget\Abp.AutoMapper.11.3.0.nupkg`.

> Keep the version aligned with the rest of your `Abp.*` packages. `Abp.AutoMapper` has a project reference to `Abp`, which becomes a NuGet dependency at the same version. An 11.3.0 `Abp.AutoMapper` next to an 11.1.0 `Abp` will either fail to restore or produce a confusing runtime binding.

### Step 3: Register a local feed

Add a local source to `aspnet-core/NuGet.Config`, keeping the ASP.NET Zero source:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <packageSources>
        <add key="ASP.NET Zero NuGet Source"
             value="https://nuget.aspnetzero.com/<YOUR-API-KEY>/v3/index.json" />
        <add key="Local" value="D:\LocalNuget" />
    </packageSources>
</configuration>
```

A folder source is fine for one developer. For a team, push to your internal feed so CI and every workstation resolve the same artifact:

```powershell
dotnet nuget push D:\LocalNuget\Abp.AutoMapper.11.3.0.nupkg `
    --source https://your-company-feed/v3/index.json `
    --api-key <KEY>
```

> **Do not push a rebuilt `Abp.AutoMapper` to nuget.org.** The package ID belongs to the ABP maintainers, and a same-ID package on a public feed causes restore conflicts that are miserable to diagnose. Keep it internal.

### Step 4 (licensed path only): patch for AutoMapper 15/16

The stock source will not compile against AutoMapper 15+. Two breaking changes hit `AbpAutoMapperModule` directly:

- The single-argument `MapperConfiguration(Action<IMapperConfigurationExpression>)` constructor is gone. The replacement is `MapperConfiguration(MapperConfigurationExpression configurationExpression, ILoggerFactory loggerFactory)`.
- A license key is expected, set through `cfg.LicenseKey`.

Bump the references in `src/Abp.AutoMapper/Abp.AutoMapper.csproj`:

```xml
<PackageReference Include="AutoMapper" Version="16.2.0" />
<PackageReference Include="AutoMapper.Collection" Version="13.0.0" />
```

The stock `CreateMappings` in `src/Abp.AutoMapper/AutoMapper/AbpAutoMapperModule.cs` looks like this:

```csharp
private void CreateMappings()
{
    Action<IMapperConfigurationExpression> configurer = configuration =>
    {
        FindAndAutoMapTypes(configuration);
        foreach (var configurator in Configuration.Modules.AbpAutoMapper().Configurators)
        {
            configurator(configuration);
        }
    };

    var config = new MapperConfiguration(configurer);   // <-- removed in AutoMapper 15

    IocManager.IocContainer.Register(
        Component.For<IConfigurationProvider>().Instance(config).LifestyleSingleton()
    );

    var mapper = config.CreateMapper();
    IocManager.IocContainer.Register(
        Component.For<IMapper>().Instance(mapper).LifestyleSingleton()
    );

#pragma warning disable CS0618
    AbpEmulateAutoMapper.Mapper = mapper;
#pragma warning restore CS0618
}
```

Rewrite it to build the expression explicitly and pass an `ILoggerFactory`:

```csharp
private void CreateMappings()
{
    var expression = new MapperConfigurationExpression();
    expression.LicenseKey = Configuration.Modules.AbpAutoMapper().LicenseKey;

    FindAndAutoMapTypes(expression);
    foreach (var configurator in Configuration.Modules.AbpAutoMapper().Configurators)
    {
        configurator(expression);
    }

    var loggerFactory = IocManager.IsRegistered<ILoggerFactory>()
        ? IocManager.Resolve<ILoggerFactory>()
        : NullLoggerFactory.Instance;

    var config = new MapperConfiguration(expression, loggerFactory);

    IocManager.IocContainer.Register(
        Component.For<AutoMapper.IConfigurationProvider>().Instance(config).LifestyleSingleton()
    );

    var mapper = config.CreateMapper();
    IocManager.IocContainer.Register(
        Component.For<IMapper>().Instance(mapper).LifestyleSingleton()
    );

#pragma warning disable CS0618
    AbpEmulateAutoMapper.Mapper = mapper;   // keep this line
#pragma warning restore CS0618
}
```

> **Do not drop the `AbpEmulateAutoMapper.Mapper` assignment.** That static field is what backs the legacy `.MapTo<T>()` extension methods in `AutoMapExtensions`, which older ASP.NET Zero solutions use heavily. Removing it compiles cleanly and then throws `NullReferenceException` the first time any `.MapTo<T>()` call runs.

`ILoggerFactory` and `NullLoggerFactory` come from `Microsoft.Extensions.Logging` and `Microsoft.Extensions.Logging.Abstractions`. In an ASP.NET Core host, `services.AddAbp()` bridges the service collection into Windsor so `ILoggerFactory` resolves; the fallback keeps unit-test hosts and the MAUI project working.

Add the `LicenseKey` property to `IAbpAutoMapperConfiguration` and `AbpAutoMapperConfiguration`:

```csharp
public interface IAbpAutoMapperConfiguration
{
    List<Action<IMapperConfigurationExpression>> Configurators { get; }

    string LicenseKey { get; set; }
}
```

Applications then set it in their own `PreInitialize`, reading it from `appsettings.json`:

```csharp
Configuration.Modules.AbpAutoMapper().LicenseKey =
    IocManager.Resolve<IConfiguration>()["AutoMapper:LicenseKey"];
```

Keeping the key out of the package matters: a license key baked into a NuGet artifact you rebuild every ABP release is a key you will eventually leak. Pack and publish as in steps 2 and 3, and consider a distinct package ID such as `YourCompany.Abp.AutoMapper` now that you have diverged from upstream; it makes the provenance obvious to whoever reads the `.csproj` next.

With the package in place, the revert itself is eight steps.

## Step 1: Swap the Package References

Four project files reference `Abp.Mapperly`: `.Core`, `.Web.Core`, `.Web.Mvc` and `.Maui`. In each one:

```xml
<!-- Remove -->
<PackageReference Include="Abp.Mapperly" Version="11.3.0" />

<!-- Add -->
<PackageReference Include="Abp.AutoMapper" Version="11.3.0" />
```

There is no separate `Riok.Mapperly` reference to clean up: the source generator arrives transitively through `Abp.Mapperly` and leaves with it.

## Step 2: Swap the Module Dependencies

**`AbpZeroTemplateCoreModule`** in the `.Core` project:

```csharp
using Abp.AutoMapper;   // was: using Abp.Mapperly;

#pragma warning disable CS0618 // AbpAutoMapperModule is obsolete - intentional, see docs/mapping.md
[DependsOn(
    typeof(AbpZeroTemplateCoreSharedModule),
    typeof(AbpZeroCoreModule),
    typeof(AbpZeroLdapModule),
    typeof(AbpAutoMapperModule),   // was: AbpMapperlyModule
    typeof(AbpAspNetZeroCoreModule),
    typeof(AbpMailKitModule),
    typeof(AbpZeroCoreOpenIddictModule))]
#pragma warning restore CS0618
public class AbpZeroTemplateCoreModule : AbpModule
```

**`AbpZeroTemplateMauiModule`**, if you build the mobile app:

```csharp
using Abp.AutoMapper;   // was: using Abp.Mapperly;

[DependsOn(typeof(AbpZeroTemplateClientModule), typeof(AbpAutoMapperModule))]
public class AbpZeroTemplateMauiModule : AbpModule
```

> Both `AbpAutoMapperModule` and `AbpMapperlyModule` call `Configuration.ReplaceService<IObjectMapper, ...>()` in `PreInitialize`. If both are in the graph, whichever runs **last** silently wins and the other's mappings fail at runtime with "no mapping found". Never leave both in place, even temporarily.

## Step 3: Restore `CustomDtoMapper.cs` in the Application Project

You do not write this from scratch; it is in your own git history. Find the commit before you merged the v15.2 upgrade:

```powershell
git log --oneline --all -- "**/CustomDtoMapper.cs"
```

Then restore the file:

```powershell
git checkout <commit> -- `
    aspnet-core/src/MyCompanyName.AbpZeroTemplate.Application/CustomDtoMapper.cs
```

> Use `git checkout`, not `git show ... > file`. In Windows PowerShell 5.1, `>` writes UTF-16LE, and the C# compiler will not read the result as a source file.

The recovered file is the template's mapping configuration plus everything your team added under the `/* ADD YOUR OWN CUSTOM AUTOMAPPER MAPPINGS HERE */` marker. It looks like this:

```csharp
internal static class CustomDtoMapper
{
    public static void CreateMappings(IMapperConfigurationExpression configuration)
    {
        //Inputs
        configuration.CreateMap<CheckboxInputType, FeatureInputTypeDto>();
        configuration.CreateMap<IInputType, FeatureInputTypeDto>()
            .Include<CheckboxInputType, FeatureInputTypeDto>()
            .Include<SingleLineStringInputType, FeatureInputTypeDto>()
            .Include<ComboboxInputType, FeatureInputTypeDto>();

        //Role
        configuration.CreateMap<RoleEditDto, Role>().ReverseMap();
        configuration.CreateMap<Role, RoleListDto>();

        //User
        configuration.CreateMap<User, UserEditDto>()
            .ForMember(dto => dto.Password, options => options.Ignore())
            .ReverseMap()
            .ForMember(user => user.Password, options => options.Ignore());

        //Webhooks
        configuration.CreateMap<WebhookSendAttempt, GetAllSendAttemptsOutput>()
            .ForMember(dto => dto.WebhookName, options => options.MapFrom(l => l.WebhookEvent.WebhookName))
            .ForMember(dto => dto.Data, options => options.MapFrom(l => l.WebhookEvent.Data));

        // ... ~120 more mappings ...

        /* ADD YOUR OWN CUSTOM AUTOMAPPER MAPPINGS HERE */
    }
}
```

Re-register it in `AbpZeroTemplateApplicationModule.PreInitialize`:

```csharp
public override void PreInitialize()
{
    //Adding authorization providers
    Configuration.Authorization.Providers.Add<AppAuthorizationProvider>();

    //Adding custom AutoMapper configuration
    Configuration.Modules.AbpAutoMapper().Configurators.Add(CustomDtoMapper.CreateMappings);
}
```

## Step 4: Restore the GraphQL Layer

The GraphQL project needs four things restored. Skip this step entirely if your solution does not include it.

**4a. Its own `CustomDtoMapper.cs`**, at `aspnet-core/src/MyCompanyName.AbpZeroTemplate.GraphQL/Startup/CustomDtoMapper.cs`:

```csharp
using AutoMapper;
using MyCompanyName.AbpZeroTemplate.Authorization.Users;
using MyCompanyName.AbpZeroTemplate.Dto;

namespace MyCompanyName.AbpZeroTemplate.Startup;

public static class CustomDtoMapper
{
    public static void CreateMappings(IMapperConfigurationExpression configuration)
    {
        configuration.CreateMap<User, UserDto>()
            .ForMember(dto => dto.Roles, options => options.Ignore())
            .ForMember(dto => dto.OrganizationUnits, options => options.Ignore());
    }
}
```

**4b. The registration** in `AbpZeroTemplateGraphQLModule`:

```csharp
public override void PreInitialize()
{
    base.PreInitialize();

    //Adding custom AutoMapper configuration
    Configuration.Modules.AbpAutoMapper().Configurators.Add(CustomDtoMapper.CreateMappings);
}
```

**4c. The projection helpers** in `Core/Base/AbpZeroTemplateQueryBase.cs`. This is where reverting actually buys you something: with `Abp.AutoMapper`, `IObjectMapper.ProjectTo` works for *any* configured `CreateMap`, so you can push projections back into SQL instead of materializing entities first.

```csharp
using Microsoft.EntityFrameworkCore;

public abstract class AbpZeroTemplateQueryBase<TField, TResult> : ITransientDependency
{
    public IObjectMapper ObjectMapper { protected get; set; }

    // ...

    public IQueryable<TDestination> ProjectTo<TDestination>(IQueryable source)
    {
        return ObjectMapper.ProjectTo<TDestination>(source);
    }

    public async Task<List<TDestination>> ProjectToListAsync<TDestination>(IQueryable source)
    {
        return await ProjectTo<TDestination>(source).ToListAsync();
    }
}
```

> The pre-v15.2 template injected AutoMapper's `IMapper` here directly. Going through `IObjectMapper` is equivalent: `AutoMapperObjectMapper.ProjectTo` simply delegates to `Mapper.ProjectTo<TDestination>(source)` and keeps the GraphQL layer independent of which mapper is configured.

**4d. The queries.** `OrganizationUnitQuery` and `RoleQuery` were rewritten during the migration to materialize before mapping, because Mapperly could not project them. Put the projections back:

```csharp
// OrganizationUnitQuery.cs
- var organizationUnits = await query.ToListAsync();
- return ObjectMapper.Map<List<OrganizationUnitDto>>(organizationUnits);
+ return await ProjectToListAsync<OrganizationUnitDto>(query);

// RoleQuery.cs
- var roles = await query.ToListAsync();
- return ObjectMapper.Map<List<RoleDto>>(roles);
+ return await ProjectToListAsync<RoleDto>(query);
```

In `UserQuery.cs`, the `GetRolesOfUsers` helper also gained a `.ToList()` that was added only because Mapperly cannot map an `IQueryable` directly. AutoMapper can, so you may drop it:

```csharp
- var roles = _roleManager.Roles.Where(x => roleIds.Contains(x.Id)).ToList();
+ var roles = _roleManager.Roles.Where(x => roleIds.Contains(x.Id));
  return ObjectMapper.Map<List<UserDto.RoleDto>>(roles);
```

This one is best practice, not a requirement; leaving the `.ToList()` in place works fine under AutoMapper too.

## Step 5: Restore the `[AutoMap*]` Attributes

Twenty-eight files lost their attributes. Recover each from your pre-upgrade branch; the fastest route is a targeted checkout:

```powershell
git checkout <pre-upgrade-commit> -- `
    aspnet-core/src/MyCompanyName.AbpZeroTemplate.Core/Friendships/Cache/FriendCacheItem.cs
```

Here is the full list, so nothing is missed.

**`.Core` (1 file)**

| File | Attribute |
|---|---|
| `Friendships/Cache/FriendCacheItem.cs` | `[AutoMapFrom(typeof(Friendship))]` |

**`.Web.Core` (1 file)**

| File | Attribute |
|---|---|
| `Models/TokenAuth/ExternalLoginProviderInfoModel.cs` | `[AutoMapFrom(typeof(ExternalLoginProviderInfo))]` |

**`.GraphQL` (3 files)**

| File | Attribute |
|---|---|
| `Dto/OrganizationUnitDto.cs` | `[AutoMapFrom(typeof(OrganizationUnit))]` |
| `Dto/RoleDto.cs` | `[AutoMapFrom(typeof(Role))]` |
| `Dto/UserDto.cs` | `[AutoMapFrom(typeof(Role))]` on nested `RoleDto`, `[AutoMapFrom(typeof(OrganizationUnit))]` on nested `OrganizationUnitDto` |

**`.Maui` (10 files)**

| File | Attribute |
|---|---|
| `Models/Common/ApplicationInfoPersistanceModel.cs` | `[AutoMapFrom(typeof(ApplicationInfoDto)), AutoMapTo(typeof(ApplicationInfoDto))]` |
| `Models/Common/AuthenticateResultPersistanceModel.cs` | `[AutoMapFrom(typeof(AbpAuthenticateResultModel)), AutoMapTo(...)]` |
| `Models/Common/CurrentLoginInformationPersistanceModel.cs` | `[AutoMapFrom(typeof(GetCurrentLoginInformationsOutput)), AutoMapTo(...)]` |
| `Models/Common/TenantInformationPersistanceModel.cs` | `[AutoMapFrom(typeof(TenantInformation)), AutoMapTo(...)]` |
| `Models/Common/TenantLoginInfoPersistanceModel.cs` | `[AutoMapFrom(typeof(TenantLoginInfoDto)), AutoMapTo(...)]` |
| `Models/Common/UserLoginInfoPersistanceModel.cs` | `[AutoMapFrom(typeof(UserLoginInfoDto)), AutoMapTo(...)]` |
| `Models/Tenants/EditTenantModel.cs` | `[AutoMap(typeof(TenantEditDto))]` |
| `Models/User/CreateOrEditUserModel.cs` | `[AutoMap(typeof(CreateOrUpdateUserInput), typeof(UserEditDto))]` |
| `Models/User/OrganizationUnitModel.cs` | `[AutoMapFrom(typeof(OrganizationUnitDto))]` |
| `Models/User/UserListModel.cs` | `[AutoMapFrom(typeof(UserListDto))]` |

**`.Web.Mvc` (13 files)**

| File | Attribute |
|---|---|
| `Areas/AppAreaName/Models/Editions/EditEditionModalViewModel.cs` | `[AutoMapFrom(typeof(GetEditionEditOutput))]` |
| `Areas/AppAreaName/Models/EntityChanges/EntityChangeListViewModel.cs` | `[AutoMapFrom(typeof(EntityAndPropertyChangeListDto))]` |
| `Areas/AppAreaName/Models/Languages/CreateOrEditLanguageModalViewModel.cs` | `[AutoMapFrom(typeof(GetLanguageForEditOutput))]` |
| `Areas/AppAreaName/Models/OrganizationUnits/EditOrganizationUnitModalViewModel.cs` | `[AutoMapFrom(typeof(OrganizationUnit))]` |
| `Areas/AppAreaName/Models/Profile/MySettingsViewModel.cs` | `[AutoMapFrom(typeof(CurrentUserProfileEditDto))]` |
| `Areas/AppAreaName/Models/Roles/CreateOrEditRoleModalViewModel.cs` | `[AutoMapFrom(typeof(GetRoleForEditOutput))]` |
| `Areas/AppAreaName/Models/SubscriptionManagement/ShowDetailModalViewModel.cs` | `[AutoMapFrom(typeof(SubscriptionPaymentProductDto))]` |
| `Areas/AppAreaName/Models/Tenants/TenantFeaturesEditViewModel.cs` | `[AutoMapFrom(typeof(GetTenantFeaturesEditOutput))]` |
| `Areas/AppAreaName/Models/Users/CreateOrEditUserModalViewModel.cs` | `[AutoMapFrom(typeof(GetUserForEditOutput))]` |
| `Areas/AppAreaName/Models/Users/UserPermissionsEditViewModel.cs` | `[AutoMapFrom(typeof(GetUserPermissionsForEditOutput))]` |
| `Models/TenantRegistration/EditionsSelectViewModel.cs` | `[AutoMapFrom(typeof(EditionsSelectOutput))]` |
| `Models/TenantRegistration/TenantRegisterResultViewModel.cs` | `[AutoMapFrom(typeof(RegisterTenantOutput))]` |
| `Views/Shared/Components/TenantChange/TenantChangeViewModel.cs` | `[AutoMapFrom(typeof(GetCurrentLoginInformationsOutput))]` |

Each file also needs its `using Abp.AutoMapper;` back, and the MAUI and GraphQL files need the DTO namespace imports that were removed alongside the attributes.

If you would rather not spread attributes across projects again, every one of these can instead be expressed as a `CreateMap` line in `CustomDtoMapper`. Functionally identical: pick one approach and apply it consistently.

## Step 6: Reconcile Against the Newer Template

Your restored `CustomDtoMapper.cs` covers the type pairs that existed at *your previous version*. Any DTO the template added between then and the version you upgraded to has no mapping, and AutoMapper will not tell you at compile time.

Walk the `Mappers/` folders in the upgraded template and add a `CreateMap` for every `[Mapper]` class you do not already cover.

| Mapperly | AutoMapper |
|---|---|
| `MapperBase<TSource, TDestination>` | `CreateMap<TSource, TDestination>()` |
| `TwoWayMapperBase<TSource, TDestination>` | `CreateMap<TSource, TDestination>().ReverseMap()` |
| `ProjectionMapperBase<TSource, TDestination>` | `CreateMap<TSource, TDestination>()` — `ProjectTo` needs no extra declaration |
| `[MapperIgnoreTarget(nameof(Dto.Password))]` | `.ForMember(d => d.Password, o => o.Ignore())` |
| `[MapperIgnoreSource(nameof(Entity.Password))]` | `.ForSourceMember(s => s.Password, o => o.DoNotValidate())` |
| `[MapProperty(nameof(User.EmailAddress), nameof(UserDto.Email))]` | `.ForMember(d => d.Email, o => o.MapFrom(s => s.EmailAddress))` |
| `AfterMap` / `BeforeMap` override | `.AfterMap((src, dest) => ...)` / `.BeforeMap(...)` |
| `IAbpMapperlyMultiLingualMapper<...>` | `configuration.CreateMultiLingualMap<TEntity, TTranslation, TDto>(context)` |

A worked example: this Mapperly class

```csharp
[Mapper]
public partial class UserToUserEditDtoMapper : MapperBase<User, UserEditDto>
{
    [MapperIgnoreTarget(nameof(UserEditDto.Password))]
    public override partial UserEditDto Map(User source);

    [MapperIgnoreTarget(nameof(UserEditDto.Password))]
    public override partial void Map(User source, UserEditDto destination);
}
```

becomes this single line:

```csharp
configuration.CreateMap<User, UserEditDto>()
    .ForMember(dto => dto.Password, options => options.Ignore());
```

Note the asymmetry in effort. The pre-v15.2 template expressed the same coverage in **67 `CreateMap` calls plus `[AutoMap*]` attributes on 28 files** against 114 Mapperly classes today. `ReverseMap()` and `Include<>()` collapse pairs and inheritance chains that Mapperly has to spell out one at a time, so the AutoMapper form is materially more compact. Going back is less work than going forward was.

**Multi-lingual entities need attention.** If you use `IMultiLingualEntity`, the Mapperly side expresses translation selection through `IAbpMapperlyMultiLingualMapper<...>`; the AutoMapper side uses `CreateMultiLingualMap`, which hooks `BeforeMap` to pick the translation for the current UI culture with fallback:

```csharp
configuration.CreateMultiLingualMap<Product, ProductTranslation, ProductListDto>(
    new MultiLingualMapContext(settingManager),
    fallbackToParentCultures: true);
```

A missing multi-lingual map degrades quietly to untranslated output rather than throwing, so verify these explicitly rather than assuming.

## Step 7: Delete the Mapperly Artifacts

Delete the `Mappers/` folders from all six projects:

```powershell
Remove-Item -Recurse -Force `
    aspnet-core/src/MyCompanyName.AbpZeroTemplate.Application/Mappers, `
    aspnet-core/src/MyCompanyName.AbpZeroTemplate.Core/Mappers, `
    aspnet-core/src/MyCompanyName.AbpZeroTemplate.Web.Core/Mappers, `
    aspnet-core/src/MyCompanyName.AbpZeroTemplate.Web.Mvc/Mappers, `
    aspnet-core/src/MyCompanyName.AbpZeroTemplate.GraphQL/Mappers, `
    aspnet-core/src/MyCompanyName.AbpZeroTemplate.Maui/Mappers
```

Then drop the Mapperly analyzer suppressions from `aspnet-core/common.props`:

```xml
<Project>
  <PropertyGroup>
    <Version>15.4.0</Version>
    <!-- Removed: <NoWarn>$(NoWarn);RMG020;RMG012;RMG066</NoWarn> -->
  </PropertyGroup>
</Project>
```

Those were Mapperly diagnostics: *source member is not mapped to any target member* (RMG020), *source member was not found for target member* (RMG012), and *no members are mapped in an object mapping* (RMG066). With the generator gone they are dead configuration, and leaving them behind hides the fact that the suppressions were ever needed.

## Step 8: Build, Validate, Test

```powershell
dotnet build aspnet-core/MyCompanyName.AbpZeroTemplate.Web.sln
```

A clean build is necessary but **not** sufficient. This is the central trade-off you are accepting: Mapperly turns a missing mapping into a compile error; AutoMapper turns it into a runtime `AutoMapperMappingException` on whichever request happens to need it.

Restore the safety net with a configuration validation test:

```csharp
public class AutoMapper_Tests : AppTestBase
{
    [Fact]
    public void AutoMapper_Configuration_Should_Be_Valid()
    {
        Resolve<AutoMapper.IConfigurationProvider>().AssertConfigurationIsValid();
    }
}
```

`AbpAutoMapperModule` registers `IConfigurationProvider` as a singleton for exactly this. Qualify the type name: `AutoMapper.IConfigurationProvider` collides with `Microsoft.Extensions.Configuration.IConfigurationProvider`, and both namespaces are usually in scope in an ASP.NET Zero test project.

`AssertConfigurationIsValid()` catches unmapped destination members across every configured pair, which is precisely the class of bug a partial revert introduces. Make it part of CI, not a one-off local run.

Then run the existing suite and smoke-test the areas with the densest mapping:

```powershell
dotnet test aspnet-core/test/MyCompanyName.AbpZeroTemplate.Tests
dotnet test aspnet-core/test/MyCompanyName.AbpZeroTemplate.GraphQL.Tests
```

Then walk this manual verification checklist; these areas exercise the mappings most likely to have been missed:

- **Users**: list, create, edit, permissions modal, Excel export, user import
- **Roles**: list, create/edit modal with permissions
- **Tenants**: list, create/edit, features modal, tenant registration flow
- **Editions**: list, create/edit, subscription and payment history
- **Audit logs and entity change history**: list and detail modals
- **Organization units**: tree, member and role assignment
- **Dynamic properties**: definitions and per-entity values
- **Webhooks**: subscriptions and send-attempt list
- **Chat and friendships** (`FriendCacheItem`: the cache path only exercises after a cache miss)
- **My Settings / profile page**
- **GraphQL queries** for users, roles and organization units, if you use them
- **MAUI login and tenant switch**, if you build the mobile app

## Living With the Revert

Two things are now permanently on your plate.

**Every ASP.NET Zero upgrade needs the revert re-applied.** When the template adds a DTO, it adds a Mapperly mapper for it. Your merge will bring that mapper in and your build will keep passing, but the mapping will not be registered: `AbpAutoMapperModule` is what is running. Add a step to your upgrade checklist: after every merge, diff the incoming `Mappers/` folders, translate anything new into `CustomDtoMapper`, then delete the folders again. The `AssertConfigurationIsValid()` test will catch most misses, and it is far cheaper than finding them in production.

**Power Tools generates Mapperly mappers.** ASP.NET Zero Power Tools emits `[Mapper]` classes for new entities. You have two options: convert each generated mapper into `CreateMap` calls after generation, or customize the Power Tools templates so they emit AutoMapper configuration directly. For teams generating entities regularly, customizing the templates pays for itself quickly.

Also worth knowing: **Native AOT and aggressive trimming are off the table** while you are on AutoMapper, since it relies on runtime reflection and expression compilation.

## If the Full Revert Is Too Much

There is a lighter option worth considering before you commit to all of the above: leave `IObjectMapper` bound to Mapperly so the template code works exactly as shipped, and register AutoMapper's `IMapper` separately for your own application code.

```csharp
// AbpZeroTemplateApplicationModule.PostInitialize - do NOT add AbpAutoMapperModule to DependsOn
private void RegisterAutoMapper()
{
    var expression = new MapperConfigurationExpression();
    CustomDtoMapper.CreateMappings(expression);

    var config = new MapperConfiguration(expression);

    IocManager.IocContainer.Register(
        Component.For<AutoMapper.IConfigurationProvider>().Instance(config).LifestyleSingleton(),
        Component.For<IMapper>().Instance(config.CreateMapper()).LifestyleSingleton()
    );
}
```

This has to live in the `.Application` module, not `AbpZeroTemplateCoreModule`: `CustomDtoMapper` is `internal static` in the `.Application` project, and `.Core` sits below `.Application` in the dependency graph. Note also that the single-argument `new MapperConfiguration(cfg => ...)` overload used elsewhere in older samples is gone in AutoMapper 15+, which is why the expression is built explicitly here.

Because `AbpAutoMapperModule` is not in the graph, nothing scans for `[AutoMap*]` attributes in this setup. Any mapping you want on the AutoMapper side has to be a `CreateMap` call in `CustomDtoMapper`.

Inject `IMapper` where your own code needs AutoMapper, and keep using `ObjectMapper` everywhere the template does. You skip steps 2 through 7 entirely, your upgrade merges stay clean, and you keep the option of converting feature by feature later. The cost is a rule your team has to internalize: **`ObjectMapper` is Mapperly, `IMapper` is AutoMapper.**

Choose the full revert when you want one mapping technology in the solution and your code genuinely depends on AutoMapper semantics: `ITypeConverter`, `IValueResolver`, inheritance maps via `Include`, or `ProjectTo` across many type pairs. Choose the hybrid when your goal is simply to avoid rewriting your own mappings during an upgrade.

## Conclusion

Mapperly is the default from ASP.NET Zero v15.2 onward, and for new projects it is the right default. But the switch is not a one-way door: `Abp.AutoMapper` is still in the ABP source tree, still packable, and still functional. Deprecated, not deleted.

The revert comes down to this:

- Decide your AutoMapper version first. The free MIT line ends at 14.0.0 and works with `Abp.AutoMapper` unmodified; 15/16 are RPL-1.5/commercial and need the module patched for the new `MapperConfiguration` constructor and license key.
- Get `Abp.AutoMapper` from your ASP.NET Zero feed, or build it from source and publish to a local or internal feed.
- Swap four package references and two module dependencies, restore `CustomDtoMapper.cs` in two projects, restore attributes in 28 files, restore the GraphQL projection helpers, reconcile against the newer template, and delete 37 mapper files.
- Add `AssertConfigurationIsValid()` to CI. It is the thing that replaces the compile-time safety you are giving up.

If you hit a mapping scenario this guide does not cover, open a ticket on [support.aspnetzero.com](https://support.aspnetzero.com/); we are happy to work through it with you.

## Further Reading

- [Migrating from AutoMapper to Mapperly](https://docs.aspnetzero.com/aspnet-core-mvc/latest/Migrating-From-AutoMapper-To-Mapperly): the forward migration guide, useful in reverse
- [DTO Mappings](https://docs.aspnetzero.com/aspnet-core-mvc/latest/Infrastructure-Core-Mvc-Dto-Mappings): how mapping works in the current template
- [ABP Object to Object Mapping](https://aspnetboilerplate.com/Pages/Documents/Object-To-Object-Mapping): the `Abp.AutoMapper` integration, including `[AutoMap*]` attributes and custom configurators
- [AutoMapper 15.0 Upgrade Guide](https://docs.automapper.io/en/stable/15.0-Upgrade-Guide.html): the breaking changes referenced above
- [Lucky Penny Software Licensing FAQ](https://luckypennysoftware.com/faq): AutoMapper's commercial terms
- [Mapperly Documentation](https://mapperly.riok.app/docs/intro/)
