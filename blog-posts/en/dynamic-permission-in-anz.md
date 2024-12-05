# Dynamic Permission in ASP.NET Zero

In this article, we will learn how to implement dynamic permissions in ASP.NET Zero. Dynamic permission is a feature that allows you to create permissions dynamically and assign them to users or roles. This feature is useful when you want to give users the ability to create custom permissions based on their requirements.

Sample source code for this article is available on [GitHub](https://github.com/aspnetzero/aspnet-zero-samples) at `DynamicPermissionSampleDemo` folder.

## Create a New Entity for Dynamic Permissions

The first step is to create a new entity to store the dynamic permissions. Create a new class called `DynamicPermission` in the `Core` project under the `DynamicPermissions` folder and add the following code:

```csharp
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations.Schema;
using Abp.Domain.Entities;

namespace DynamicPermissionSampleDemo.DynamicPermissions;

[Table("DynamicPermissions")]
public class DynamicPermission : Entity, IMayHaveTenant
{
    public virtual string Name { get; set; }

    public virtual string Description { get; set; }

    public virtual string DisplayName { get; set; }

    public virtual int? ParentId { get; set; }

    public int? TenantId { get; set; }

    [ForeignKey("ParentId")]
    public DynamicPermission Parent { get; set; }

    public List<DynamicPermission> Children { get; set; }

    public DynamicPermission()
    {
        Children = new List<DynamicPermission>();
    }
}
```

### Update DbContext

Add this class to the `DbContext` in the `EntityFrameworkCore` project:

```csharp
public virtual DbSet<DynamicPermission> DynamicPermissions { get; set; }
```

### Apply Migration

After adding the entity to the `DbContext`, create a migration and update the database to add the `DynamicPermission` table.

## Override `PermissionManager`

### Step 1: Create `DynamicPermissionManager`

Create a new class called `DynamicPermissionManager` in the `Core` project under the `DynamicPermissions` folder. Add the following code:

```csharp
using System.Collections.Generic;
using System.Linq;
using Abp.Authorization;
using Abp.Configuration.Startup;
using Abp.Dependency;
using Abp.Domain.Repositories;
using Abp.Domain.Uow;
using Abp.Localization;

namespace DynamicPermissionSampleDemo.DynamicPermissions;

public class DynamicPermissionManager : PermissionManager
{
    private readonly IRepository<DynamicPermission, int> _repository;
    private readonly IUnitOfWorkManager _uow;

    public DynamicPermissionManager(IIocManager iocManager, IAuthorizationConfiguration authorizationConfiguration,
        IUnitOfWorkManager unitOfWorkManager, IMultiTenancyConfig multiTenancy,
        IRepository<DynamicPermission, int> repository) 
        : base(iocManager, authorizationConfiguration, unitOfWorkManager, multiTenancy)
    {
        _repository = repository;
        _uow = unitOfWorkManager;
    }

    public override void Initialize()
    {
        Permissions.Clear();

        var databasePermissions = GetPermissionsFromDatabase();
        foreach (var permission in databasePermissions)
        {
            Permissions[permission.Name] = permission;
        }

        base.Initialize();
    }

    private List<Permission> GetPermissionsFromDatabase()
    {
        using var uow = _uow.Begin();
        var dynamicPermissions = _repository.GetAllList().ToList();

        var permissions = new List<Permission>();
        foreach (var item in dynamicPermissions)
        {
            var permission = new Permission(item.Name, L(item.DisplayName), L(item.Description));

            if (!item.ParentId.HasValue)
            {
                foreach (var child in item.Children)
                {
                    permission.CreateChildPermission(child.Name, L(child.DisplayName), L(child.Description));
                }
                permissions.Add(permission);
            }
        }

        uow.Complete();
        return permissions;
    }

    private static ILocalizableString L(string name)
    {
        return new LocalizableString(name, DynamicPermissionSampleDemoConsts.LocalizationSourceName);
    }
}
```

### Step 2: Create `DynamicPermissionManagerHandler`

Create a new class called `DynamicPermissionManagerHandler` in the `Core` project under the `DynamicPermissions` folder. Add the following code:

```csharp
using System;
using System.Linq;
using Abp.Authorization;
using Castle.MicroKernel;

namespace DynamicPermissionSampleDemo.DynamicPermissions;

public class DynamicPermissionManagerHandler : IHandlerSelector
{
    public bool HasOpinionAbout(string key, Type service)
    {
        return typeof(IPermissionManager) == service;
    }

    public IHandler SelectHandler(string key, Type service, IHandler[] handlers)
    {
        return handlers.FirstOrDefault(x => x.ComponentModel.Implementation == typeof(DynamicPermissionManager));
    }
}
```

### Step 3: Update `DynamicPermissionSampleDemoCoreModule`

#### Add to `PreInitialize`

In the `DynamicPermissionSampleDemoCoreModule` class, add the following code to the `PreInitialize` method:

```csharp
IocManager.IocContainer.Kernel.AddHandlerSelector(new DynamicPermissionManagerHandler());
```

#### Add to `PostInitialize`

In the `DynamicPermissionSampleDemoCoreModule` class, add the following code to the `PostInitialize` method:

```csharp
IocManager.Resolve<DynamicPermissionManager>().Initialize();
```

## Create Application Service for Dynamic Permissions

### Step 1: Define the Interface

Create a new class called `IDynamicPermissionAppService` in the `Application.Shared` project under the `DynamicPermissions` folder and add the following code:

```csharp
using System.Threading.Tasks;
using Abp.Application.Services;
using Abp.Application.Services.Dto;
using DynamicPermissionSampleDemo.Notifications.Dto;

namespace DynamicPermissionSampleDemo.DynamicPermissions;

public interface IDynamicPermissionsAppService : IApplicationService
{
    Task<PagedResultDto<GetDynamicPermissionForViewDto>> GetAll();
    
    Task CreateOrEdit(CreateOrEditDynamicPermissionDto input);

    Task<GetDynamicPermissionForEditOutput> GetDynamicPermissionForEdit(EntityDto input);

    Task Delete(EntityDto input);

    Task<PagedResultDto<DynamicPermissionDynamicPermissionLookupTableDto>> GetAllDynamicPermissionForLookupTable(GetAllForLookupTableInput input);
}

public class GetDynamicPermissionForViewDto
{
    public DynamicPermissionDto DynamicPermission { get; set; }

    public string DynamicPermissionName { get; set; }
}

public class DynamicPermissionDto : EntityDto
{
    public string Name { get; set; }

    public string Description { get; set; }

    public string DisplayName { get; set; }

    public int? ParentId { get; set; }
}

public class CreateOrEditDynamicPermissionDto : EntityDto<int?>
{
    public string Name { get; set; }

    public string Description { get; set; }

    public string DisplayName { get; set; }

    public int? ParentId { get; set; }
}

public class GetDynamicPermissionForEditOutput
{
    public CreateOrEditDynamicPermissionDto DynamicPermission { get; set; }

    public string DynamicPermissionName { get; set; }
}

public class DynamicPermissionDynamicPermissionLookupTableDto
{
    public int Id { get; set; }

    public string DisplayName { get; set; }
}
```

> **Note:** You can separate the DTOs into individual files if needed.

### Step 2: Implement the Service

Create a new class called `DynamicPermissionsAppService` in the `Application` project under the `DynamicPermissions` folder and add the following code:

```csharp
using System.Linq;
using System.Linq.Dynamic.Core;
using Abp.Linq.Extensions;
using System.Collections.Generic;
using System.Threading.Tasks;
using Abp.Domain.Repositories;
using Abp.Application.Services.Dto;
using DynamicPermissionSampleDemo.Notifications.Dto;
using Microsoft.EntityFrameworkCore;

namespace DynamicPermissionSampleDemo.DynamicPermissions;

public class DynamicPermissionsAppService : DynamicPermissionSampleDemoAppServiceBase, IDynamicPermissionsAppService
{
    private readonly IRepository<DynamicPermission> _dynamicPermissionRepository;
    private readonly DynamicPermissionManager _permissionManager;

    public DynamicPermissionsAppService(IRepository<DynamicPermission> dynamicPermissionRepository, DynamicPermissionManager permissionManager)
    {
        _dynamicPermissionRepository = dynamicPermissionRepository;
        _permissionManager = permissionManager;
    }
    
    public async Task<PagedResultDto<GetDynamicPermissionForViewDto>> GetAll(PagedAndSortedResultRequestDto input)
    {
        var filteredDynamicPermissions = _dynamicPermissionRepository.GetAll()
            .Include(e => e.Parent);

        var pagedAndFilteredDynamicPermissions = filteredDynamicPermissions
            .OrderBy("id asc")
            .PageBy(input);

        var dynamicPermissions = from o in pagedAndFilteredDynamicPermissions
            join o1 in _dynamicPermissionRepository.GetAll() on o.ParentId equals o1.Id into j1
            from s1 in j1.DefaultIfEmpty()
            select new
            {
                o.Name,
                o.Description,
                o.DisplayName,
                Id = o.Id,
                DynamicPermissionName = s1 == null || s1.Name == null ? "" : s1.Name.ToString()
            };

        var totalCount = await filteredDynamicPermissions.CountAsync();

        var dbList = await dynamicPermissions.ToListAsync();
        var results = new List<GetDynamicPermissionForViewDto>();

        foreach (var o in dbList)
        {
            var res = new GetDynamicPermissionForViewDto()
            {
                DynamicPermission = new DynamicPermissionDto
                {
                    Name = o.Name,
                    Description = o.Description,
                    DisplayName = o.DisplayName,
                    Id = o.Id,
                },
                DynamicPermissionName = o.DynamicPermissionName
            };

            results.Add(res);
        }

        return new PagedResultDto<GetDynamicPermissionForViewDto>(
            totalCount,
            results
        );
    }

    public async Task CreateOrEdit(CreateOrEditDynamicPermissionDto input)
    {
        if (input.Id == null)
        {
            await Create(input);
        }
        else
        {
            await Update(input);
        }
        
        await UnitOfWorkManager.Current.SaveChangesAsync();
        
        _permissionManager.Initialize();
    }
    
    protected virtual async Task Create(CreateOrEditDynamicPermissionDto input)
    {
        var dynamicPermission = ObjectMapper.Map<DynamicPermission>(input);

        if (AbpSession.TenantId != null)
        {
            dynamicPermission.TenantId = AbpSession.TenantId;
        }

        await _dynamicPermissionRepository.InsertAsync(dynamicPermission);
    }
    
    protected virtual async Task Update(CreateOrEditDynamicPermissionDto input)
    {
        var dynamicPermission = await _dynamicPermissionRepository.FirstOrDefaultAsync((int)input.Id);
        ObjectMapper.Map(input, dynamicPermission);
    }

    public async Task<GetDynamicPermissionForEditOutput> GetDynamicPermissionForEdit(EntityDto input)
    {
        var dynamicPermission = await _dynamicPermissionRepository.FirstOrDefaultAsync(input.Id);

        var output = new GetDynamicPermissionForEditOutput
            { DynamicPermission = ObjectMapper.Map<CreateOrEditDynamicPermissionDto>(dynamicPermission) };

        if (output.DynamicPermission.ParentId != null)
        {
            var lookupDynamicPermission =
                await _dynamicPermissionRepository.FirstOrDefaultAsync((int)output.DynamicPermission
                    .ParentId);
            output.DynamicPermissionName = lookupDynamicPermission?.Name;
        }

        return output;
    }

    public async Task Delete(EntityDto input)
    {
        await _dynamicPermissionRepository.DeleteAsync(input.Id);
        
        await UnitOfWorkManager.Current.SaveChangesAsync();
        
        _permissionManager.Initialize();
    }

    public async Task<PagedResultDto<DynamicPermissionDynamicPermissionLookupTableDto>> GetAllDynamicPermissionForLookupTable(GetAllForLookupTableInput input)
    {
        var query = _dynamicPermissionRepository.GetAll().WhereIf(
            !string.IsNullOrWhiteSpace(input.Filter),
            e => e.Name != null && e.Name.Contains(input.Filter)
        );

        var totalCount = await query.CountAsync();

        var dynamicPermissionList = await query
            .PageBy(input)
            .ToListAsync();

        var lookupTableDtoList = new List<DynamicPermissionDynamicPermissionLookupTableDto>();
        foreach (var dynamicPermission in dynamicPermissionList)
        {
            lookupTableDtoList.Add(new DynamicPermissionDynamicPermissionLookupTableDto
            {
                Id = dynamicPermission.Id,
                DisplayName = dynamicPermission.Name?.ToString()
            });
        }

        return new PagedResultDto<DynamicPermissionDynamicPermissionLookupTableDto>(
            totalCount,
            lookupTableDtoList
        );
    }
}
```

> **Note:** For brevity, the interface implementation is not included here but can be extended as needed. You may also refer to the sample project for additional details.

For example, you can create a new dynamic permission using the following code.

```js
abp.auth.hasPermission('Pages.Administration.Users.Unlock' + user.department)
```

This code will check if the user has the permission to unlock the user based on the department. If the user has the permission, the code will return `true`, otherwise `false`. You can create many departments and assign different permissions to each department.

## Conclusion

In this article, we learned how to implement dynamic permissions in ASP.NET Zero. Dynamic permissions allow you to create custom permissions based on your requirements. This feature is useful when you want to give users the ability to create custom permissions dynamically. You can extend this feature further by adding more functionality to the dynamic permissions.