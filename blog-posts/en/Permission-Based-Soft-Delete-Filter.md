# Implementing Permission-Based Soft Delete Filter in ASP.NET Zero

## Introduction
Soft delete is a frequently used feature in applications that can be used to mark data as deleted without removing it from the database. In some cases, users may have to switch between views that show or hide soft deleted data. In this blog post, we looked at how to implement a permission-based soft delete filter in an ASP.NET Zero application. We also showed how to add a switch component to the user interface to activate or deactivate the soft delete filter based on user permissions.

## Soft Delete in ASP.NET Zero
ASP.NET Zero is based on the ASP.NET Boilerplate framework, which contains a preconfigured soft delete mechanism. Soft delete allows data records to remain in the database while being marked as deleted. However, in some cases, administrators or authorized users may have to display these soft deleted data records.

## Permissions and Authorization for the Soft Delete Filter
Before allowing users to toggle the soft delete filter, we need to make sure they have the appropriate permissions. In ASP.NET Zero, permissions can be easily managed through the authorization system. You can create a new permission (such as **SoftDeleteFilterToggle**) and assign it to the required roles.


```c#
public static class AppPermissions
{
    //
    public const string SoftDeleteFilterToggle = "Pages.SoftDeleteFilterToggle";
}
```

```c#
public class AppAuthorizationProvider : AuthorizationProvider
{
    //

    public override void SetPermissions(IPermissionDefinitionContext context)
    {
        var administration = context.GetPermissionOrNull(AppPermissions.Pages_Administration) ?? context.CreatePermission(AppPermissions.Pages_Administration, L("Administration"));

        // Register the soft delete filter toggle permission
        administration.CreateChildPermission(AppPermissions.SoftDeleteFilterToggle, L("SoftDeleteFilterToggle"));
    }

    //
}
```

Once this permission is created and added to the appropriate user roles, only authorized users will be able to toggle the soft delete filter.

## Adding the Soft Delete Toggle Component
Similar to how organization unit switching is implemented, we can add a toggle switch component to the right-hand corner of the app. This switch will be visible only to users who have the `SoftDeleteFilterToggle` permission.

To implement this, we’ll create a new View Component called `AppSoftDeleteFilter`. The view component will be placed under `Areas/App/Views/Shared/Components/AppSoftDeleteFilter`. This folder will contain two files: `AppSoftDeleteFilterViewComponent.cs` and Default.cshtml.

### AppSoftDeleteFilterViewComponent.cs
This class will handle the logic for checking the user’s permission and determining whether to display the toggle switch.

```c#
using Microsoft.AspNetCore.Mvc;
using SoftDeleteFilterDemo.Web.Views;
using System.Threading.Tasks;

namespace SoftDeleteFilterDemo.Web.Areas.App.Views.Shared.Components.AppSoftDeleteFilter
{
    public class AppSoftDeleteFilterViewComponent : SoftDeleteFilterDemoViewComponent
    {
        public Task<IViewComponentResult> InvokeAsync()
        {
            return Task.FromResult<IViewComponentResult>(View());
        }
    }
}
```

### Default.cshtml
This Razor view file contains the markup for the toggle switch component. The toggle will appear only if the user has permission, which is determined by the logic in the view component.

```html
@using SoftDeleteFilterDemo.Authorization
@{
    Layout = null;
}

@if (PermissionChecker.IsGranted(AppPermissions.SoftDeleteFilterToggle))
{
    <span class="form-check form-switch form-check-custom">
        <label class="d-flex align-items-center">
            <input type="checkbox" id="SoftDeleteFilterToggle" class="form-check-input">
            <span class="form-check-label ms-2">
                Show Deleted Records
            </span>
        </label>
    </span>
}
```

After creating the files and their contents, navigate to the `_Layout.cshtml` files under the `Areas/App/Views/Layout` folder and add the ViewComponent as shown below. The placement of the **SoftDeleteFilter** toggle will depend on where you want it to appear in your layout:

```html
<!--begin::Navbar-->
<div class="app-navbar flex-shrink-0">
    @await Component.InvokeAsync(typeof(AppSoftDeleteFilterViewComponent))
    //
</div>
```

## Implementing the Filter Logic
ASP.NET Boilerplate provides built-in support for data filters, including the soft delete filter. You can enable or disable the soft delete filter at runtime based on the user’s choice from the switch component. If you’d like to explore other filters, you can check out the [documentation here](https://aspnetboilerplate.com/Pages/Documents/Data-Filters.)

### Implementing Soft Delete Filter Using Cache and Singleton Pattern
Explains how to manage the ability for users to enable or disable the soft delete filter using a cache structure and a singleton pattern. By utilizing this structure, the state of the soft delete filter can be managed individually for each user.

> Note: These classes need to be created under the Core folder in your project to ensure access to the DbContext.

#### SoftDeleteFilterCacheItem.cs
This class represents the cache item used to store the soft delete filter state (enabled/disabled) for each user.

```c#
public class SoftDeleteFilterCacheItem
{
    public const string CacheName = "AppSoftDeleteFilterCache";
    public bool IsSoftDeleteFilterEnabled { get; set; }
    public SoftDeleteFilterCacheItem()
    {
        
    }

    public SoftDeleteFilterCacheItem(bool isSoftDeleteFilterEnabled)
    {
        IsSoftDeleteFilterEnabled = isSoftDeleteFilterEnabled;            
    }
}
```

#### SoftDeleteFilterCacheExtensions.cs
This extension method simplifies retrieving the soft delete filter cache using `ICacheManager`.

```c#
public static class SoftDeleteFilterCacheExtensions
{
    public static ITypedCache<string, SoftDeleteFilterCacheItem> GetSoftDeleteFilterCache(this ICacheManager cacheManager)
    {
        return cacheManager.GetCache<string, SoftDeleteFilterCacheItem>(SoftDeleteFilterCacheItem.CacheName);
    }
}
```


#### CurrentUserSoftDeleteFilterProvider.cs
This class provides methods to retrieve and update the soft delete filter setting for the current user, utilizing a singleton pattern for thread safety.

```c#
public class CurrentUserSoftDeleteFilterProvider
{
    private static CurrentUserSoftDeleteFilterProvider _instance;
    private static readonly object _lock = new object();
    private readonly ICacheManager _cacheManager;
    private readonly IAbpSession _session;

    private CurrentUserSoftDeleteFilterProvider(ICacheManager cacheManager, IAbpSession session)
    {
        _cacheManager = cacheManager;
        _session = session;
    }

    public static CurrentUserSoftDeleteFilterProvider GetInstance(ICacheManager cacheManager, IAbpSession session)
    {
        lock (_lock)
        {
            if (_instance == null)
            {
                _instance = new CurrentUserSoftDeleteFilterProvider(cacheManager, session);
            }
        }
        return _instance;
    }

    public async Task<bool> IsSoftDeleteFilterEnabled()
    {
        var userIdentifier = _session?.ToUserIdentifier()?.ToString();

        if (userIdentifier != null)
        {
            var cacheItem = await _cacheManager.GetSoftDeleteFilterCache().GetOrDefaultAsync(userIdentifier);

            return cacheItem?.IsSoftDeleteFilterEnabled ?? true;
        }

        return true;
    }

    public async Task UpdateSoftDeleteFilterEnabled(bool isSoftDeleteFilterEnabled)
    {
        await _cacheManager.GetSoftDeleteFilterCache().SetAsync(_session.ToUserIdentifier().ToString(), new SoftDeleteFilterCacheItem
        {
            IsSoftDeleteFilterEnabled = isSoftDeleteFilterEnabled
        });
    }
}
```


### SoftDeleteFilterDemoDbContext.cs
This class overrides the `IsSoftDeleteFilterEnabled` property to dynamically determine the filter's state at runtime, based on the current user’s soft delete filter settings.

```c#
public class SoftDeleteFilterDemoDbContext : AbpZeroDbContext<Tenant, Role, User, SoftDeleteFilterDemoDbContext>, IOpenIddictDbContext
{
    //

    public ICacheManager CacheManager { get; set; }
    public IAbpSession Session { get; set; }

    public override bool IsSoftDeleteFilterEnabled => AsyncHelper.RunSync(() => CurrentUserSoftDeleteFilterProvider
                                                                                .GetInstance(CacheManager, Session).IsSoftDeleteFilterEnabled());

    //
}
```

### SoftDeleteFilterDto.cs
A data transfer object (DTO) that indicates whether the soft delete filter is enabled for the user.

```c#
 public class SoftDeleteFilterDto
 {
     public bool IsSoftDeleteFilterEnabled { get; set; }
 }
```

### IProfileAppService.cs
An application service interface that provides functions to get and update the soft delete filter settings.

```c#
public interface IProfileAppService : IApplicationService
{
    //

    Task<SoftDeleteFilterDto> GetSoftDeleteFilterSetting();

    Task UpdateSoftDeleteFilterSetting(SoftDeleteFilterDto input);
}
```

### ProfileAppService.cs
A service class that manages user soft delete filter settings, using SettingManager to retrieve and update the settings.

```c#
public class ProfileAppService : SoftDeleteFilterDemoAppServiceBase, IProfileAppService
{
    public async Task<SoftDeleteFilterDto> GetSoftDeleteFilterSetting()
    {
        var isSoftDeleteFilterEnabled = await CurrentUserSoftDeleteFilterProvider.GetInstance(_cacheManager, AbpSession).IsSoftDeleteFilterEnabled();

        return new SoftDeleteFilterDto
        {
            IsSoftDeleteFilterEnabled = isSoftDeleteFilterEnabled
        };
    }

    public async Task UpdateSoftDeleteFilterSetting(SoftDeleteFilterDto input)
    {
        await CurrentUserSoftDeleteFilterProvider.GetInstance(_cacheManager, AbpSession).UpdateSoftDeleteFilterEnabled(input.IsSoftDeleteFilterEnabled);
    }
}
```

### SoftDeleteFilter.js
In the SoftDeleteFilter.js JavaScript file, we manage the state of the soft delete filter checkbox and synchronize it with the backend settings. When the page loads, the script retrieves the current state of the soft delete filter from the server and sets the checkbox accordingly.

When the user toggles the checkbox, the new state is sent to the backend to update the filter settings. 

```javascript
(function ($) {
  $(function () {
    var _profileService = abp.services.app.profile;

    const checkbox = document.getElementById('SoftDeleteFilterToggle');

    _profileService.getSoftDeleteFilterSetting().done(function (result) {
      // Set the checkbox to the opposite of the received value.
      checkbox.checked = !result.isSoftDeleteFilterEnabled;
    });

    checkbox.addEventListener('change', function () {
      saveFilterState(checkbox.checked);
    });

    function saveFilterState(isChecked) {
      _profileService.updateSoftDeleteFilterSetting({
        // Invert the checkbox value before sending it to the server,
        // because we store the opposite state (e.g., if checked, we disable the filter)
        isSoftDeleteFilterEnabled: !isChecked
      }).done(function () {
        window.location.reload();
        console.log('Checkbox state updated successfully', isChecked);
      }).fail(function (error) {
        console.error('Failed to update checkbox state:', error);
      });
    }

  });
})(jQuery);
```

## Conclusion
We have enabled users to choose whether to view or hide deleted data and ensured that this feature is only available to authorized users. With the **SoftDelete** filter, it is now possible to hide or show deleted records.