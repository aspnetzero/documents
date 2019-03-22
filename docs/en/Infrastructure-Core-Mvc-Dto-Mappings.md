# DTO Mappings

ASP.NET Zero uses [AutoMapper](http://automapper.org/) for DTO to Entity mappings (and other types of object-to-object mappings). We use [Abp.AutoMapper](https://www.nuget.org/packages/Abp.AutoMapper) library that makes usage of AutoMapper simpler and declarative.

For instance, see the DTO class that is used to transfer a tenant editing information:

```
[AutoMap(typeof (Tenant))]
public class TenantEditDto : EntityDto
{
    [Required]
    [StringLength(Tenant.MaxTenancyNameLength)]
    public string TenancyName { get; set; }

    [Required]
    [StringLength(Tenant.MaxNameLength)]
    public string Name { get; set; }

    public bool IsActive { get; set; }
}
```

Here, **AutoMap** attribute automatically creates mapping between **TenantEditDto** and **Tenant** classes. Then we can automatically convert a Tenant object to TenantEditDto (and vice verse) object as shown below:

```
[AbpAuthorize(AppPermissions.Pages_Tenants_Edit)]
public async Task<TenantEditDto> GetTenantForEdit(EntityRequestInput input)
{
    return ObjectMapper.Map<TenantEditDto>(await TenantManager.GetByIdAsync(input.Id));
}
```

<span class="auto-style3">ObjectMapper</span> (this property comes from base class, but can be injected as IObjectMapper when you need somewhere else) is used to perform mappings.



## Custom Object Mappings

Attribute based mapping may not be sufficient in some cases. If you need to directly use Automapper API to configure your mappings, you should do it in **CustomDtoMapper** class.

See [Data Transfer Objects documentation](https://aspnetboilerplate.com/Pages/Documents/Data-Transfer-Objects) for more information on DTOs.