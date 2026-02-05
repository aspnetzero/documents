# DTO Mappings

ASP.NET Zero uses [Mapperly](https://mapperly.riok.app/) for DTO to Entity mappings (and other types of object-to-object mappings). Mapperly is a .NET source generator that generates mapping code at compile-time, providing better performance and type safety compared to runtime reflection-based mappers.

## Creating a Mapper

To create a mapping between two types, create a mapper class with the `[Mapper]` attribute. ASP.NET Zero uses per-functionality mapper classes following the naming convention `{SourceType}To{DestinationType}Mapper`.

For instance, see the mapper class that converts a Tenant entity to TenantEditDto:

```csharp
using Riok.Mapperly.Abstractions;

namespace Acme.PhoneBookDemo.MultiTenancy.Mapper;

[Mapper]
public partial class TenantToTenantEditDtoMapper
{
    public partial TenantEditDto Map(Tenant tenant);
}
```

The DTO class:

```csharp
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

Mapperly automatically generates the mapping implementation at compile-time by matching property names. ASP.NET Zero automatically discovers and registers these mapper classes.

## Using the Mapper

Use `ObjectMapper.Map<T>()` to convert objects. The ObjectMapper uses the Mapperly-generated mappers internally:

```csharp
[AbpAuthorize(AppPermissions.Pages_Tenants_Edit)]
public async Task<TenantEditDto> GetTenantForEdit(EntityRequestInput input)
{
    var tenant = await TenantManager.GetByIdAsync(input.Id);
    return ObjectMapper.Map<TenantEditDto>(tenant);
}
```

The `ObjectMapper` property is available in base classes (like `AppServiceBase`), or can be injected as `IObjectMapper` where needed.

## Collection Mapping

When you define a mapper method for a single object, Mapperly can automatically generate collection mappings:

```csharp
[Mapper]
public partial class PersonToPersonListDtoMapper
{
    public partial PersonListDto Map(Person person);
    public partial List<PersonListDto> Map(List<Person> persons);
}
```

Usage:

```csharp
return new ListResultDto<PersonListDto>(ObjectMapper.Map<List<PersonListDto>>(persons));
```

## Updating Existing Objects

To update an existing object instead of creating a new one, use `ObjectMapper.Map` with both source and target:

```csharp
public async Task EditPerson(EditPersonInput input)
{
    var person = await _personRepository.FirstOrDefaultAsync(input.Id);
    ObjectMapper.Map(input, person);
}
```

The corresponding mapper class:

```csharp
[Mapper]
public partial class EditPersonInputToPersonMapper
{
    public partial void Map(EditPersonInput input, Person person);
}
```

## Custom Property Mapping

When property names don't match, use the `[MapProperty]` attribute:

```csharp
[Mapper]
public partial class UserToUserDtoMapper
{
    [MapProperty(nameof(User.EmailAddress), nameof(UserDto.Email))]
    public partial UserDto Map(User user);
}
```

## Ignoring Properties

To ignore specific properties during mapping:

```csharp
[Mapper]
public partial class UserToUserDtoMapper
{
    [MapperIgnoreSource(nameof(User.Password))]
    [MapperIgnoreTarget(nameof(UserDto.FullName))]
    public partial UserDto Map(User user);
}
```

## Benefits of Mapperly

1. **Compile-time code generation**: No runtime reflection overhead
2. **Type safety**: Mapping errors are caught at compile time
3. **AOT compatibility**: Works with Native AOT compilation
4. **Performance**: Generated code is as fast as hand-written mapping
5. **Debuggable**: Generated code can be inspected and debugged

See [Mapperly documentation](https://mapperly.riok.app/docs/intro/) for more advanced mapping scenarios.

See [Data Transfer Objects documentation](https://aspnetboilerplate.com/Pages/Documents/Data-Transfer-Objects) for more information on DTOs.
