# Migrating from AutoMapper to Mapperly

ASP.NET Zero has migrated from AutoMapper to [Mapperly](https://mapperly.riok.app/) for object-to-object mappings. This guide helps you migrate your existing custom mappings from AutoMapper to Mapperly.

## Why Mapperly?

Mapperly offers several advantages over AutoMapper:

| Feature | AutoMapper | Mapperly |
|---------|------------|----------|
| Mapping Generation | Runtime reflection | Compile-time source generation |
| Performance | Good | Excellent (no reflection overhead) |
| Type Safety | Runtime errors | Compile-time errors |
| AOT Compatibility | Limited | Full support |
| Debugging | Harder to debug | Generated code is inspectable |
| Configuration | Centralized profiles | Per-functionality mapper classes |

## Architectural Changes

### Before: Centralized CustomDtoMapper

With AutoMapper, ASP.NET Zero used a centralized `CustomDtoMapper.cs` file containing all mapping configurations:

```csharp
// Old approach - CustomDtoMapper.cs
public static class CustomDtoMapper
{
    public static void CreateMappings(IMapperConfigurationExpression configuration)
    {
        configuration.CreateMap<Person, PersonListDto>();
        configuration.CreateMap<Person, GetPersonForEditOutput>();
        configuration.CreateMap<EditPersonInput, Person>();
        configuration.CreateMap<CreatePersonInput, Person>();
        // ... many more mappings
    }
}
```

### After: Per-Functionality Mapper Classes

With Mapperly, each mapping (or related group of mappings) gets its own mapper class:

```csharp
// New approach - PersonToPersonListDtoMapper.cs
[Mapper]
public partial class PersonToPersonListDtoMapper
{
    public partial PersonListDto Map(Person person);
    public partial List<PersonListDto> Map(List<Person> persons);
}

// PersonToGetPersonForEditOutputMapper.cs
[Mapper]
public partial class PersonToGetPersonForEditOutputMapper
{
    public partial GetPersonForEditOutput Map(Person person);
}

// EditPersonInputToPersonMapper.cs
[Mapper]
public partial class EditPersonInputToPersonMapper
{
    public partial void Map(EditPersonInput input, Person person);
}
```

**Naming Convention**: `{SourceType}To{DestinationType}Mapper`

## Migration Steps

### Step 1: Remove AutoMapper Packages

Remove the AutoMapper NuGet packages from your project:

```bash
dotnet remove package AutoMapper
dotnet remove package Abp.AutoMapper
```

### Step 2: Add Mapperly Package

Add the Mapperly NuGet package:

```bash
dotnet add package Riok.Mapperly
```

### Step 3: Convert Mapping Configurations

For each `CreateMap<TSource, TDestination>()` call in your `CustomDtoMapper.cs`:

1. Create a new mapper class file
2. Add the `[Mapper]` attribute
3. Define partial methods for the mappings

### Step 4: Continue Using ObjectMapper

The `ObjectMapper.Map<T>()` API remains the same. ASP.NET Zero automatically discovers your Mapperly mapper classes and uses them internally. You don't need to inject mapper instances into your services.

## Common Mapping Patterns

### Basic Mapping

**AutoMapper:**
```csharp
configuration.CreateMap<Person, PersonDto>();

// Usage
var dto = ObjectMapper.Map<PersonDto>(person);
```

**Mapperly:**
```csharp
// Create mapper class file: PersonToPersonDtoMapper.cs
[Mapper]
public partial class PersonToPersonDtoMapper
{
    public partial PersonDto Map(Person person);
}

// Usage - ObjectMapper still works the same way
var dto = ObjectMapper.Map<PersonDto>(person);
```

### Collection Mapping

**AutoMapper:**
```csharp
var dtos = ObjectMapper.Map<List<PersonDto>>(persons);
```

**Mapperly:**
```csharp
[Mapper]
public partial class PersonToPersonDtoMapper
{
    public partial PersonDto Map(Person person);
    public partial List<PersonDto> Map(List<Person> persons);
}

// Usage - ObjectMapper still works the same way
var dtos = ObjectMapper.Map<List<PersonDto>>(persons);
```

### Reverse Mapping

**AutoMapper:**
```csharp
configuration.CreateMap<EditPersonInput, Person>().ReverseMap();
```

**Mapperly:**
```csharp
// EditPersonInputToPersonMapper.cs
[Mapper]
public partial class EditPersonInputToPersonMapper
{
    public partial void Map(EditPersonInput input, Person person);
}

// For reverse mapping, create a separate mapper class
// PersonToEditPersonInputMapper.cs
[Mapper]
public partial class PersonToEditPersonInputMapper
{
    public partial EditPersonInput Map(Person person);
}
```

### Updating Existing Objects

**AutoMapper:**
```csharp
ObjectMapper.Map(input, existingPerson);
```

**Mapperly:**
```csharp
// Create mapper class
[Mapper]
public partial class EditPersonInputToPersonMapper
{
    public partial void Map(EditPersonInput input, Person target);
}

// Usage - ObjectMapper still works the same way
ObjectMapper.Map(input, existingPerson);
```

### Custom Property Mapping

**AutoMapper:**
```csharp
configuration.CreateMap<User, UserDto>()
    .ForMember(dest => dest.Email, opt => opt.MapFrom(src => src.EmailAddress));
```

**Mapperly:**
```csharp
[Mapper]
public partial class UserToUserDtoMapper
{
    [MapProperty(nameof(User.EmailAddress), nameof(UserDto.Email))]
    public partial UserDto Map(User user);
}
```

### Ignoring Properties

**AutoMapper:**
```csharp
configuration.CreateMap<User, UserDto>()
    .ForMember(dest => dest.Password, opt => opt.Ignore());
```

**Mapperly:**
```csharp
[Mapper]
public partial class UserToUserDtoMapper
{
    [MapperIgnoreTarget(nameof(UserDto.Password))]
    public partial UserDto Map(User user);
}
```

### Custom Conversion Logic

**AutoMapper:**
```csharp
configuration.CreateMap<Order, OrderDto>()
    .ConvertUsing<OrderToOrderDtoConverter>();
```

**Mapperly:**
```csharp
[Mapper]
public partial class OrderToOrderDtoMapper
{
    public partial OrderDto Map(Order order);

    // Custom mapping for specific properties
    private string MapStatus(OrderStatus status) => status.ToString().ToUpper();
}
```

### Conditional Mapping

**AutoMapper:**
```csharp
configuration.CreateMap<Person, PersonDto>()
    .ForMember(dest => dest.FullName, opt => opt.Condition(src => src.Name != null));
```

**Mapperly:**
```csharp
[Mapper]
public partial class PersonToPersonDtoMapper
{
    public partial PersonDto Map(Person person);

    private string? MapFullName(Person person)
        => person.Name != null ? $"{person.Name} {person.Surname}" : null;
}
```

## Auto-Discovery

ASP.NET Zero automatically discovers and registers Mapperly mapper classes. You don't need to manually register them in the dependency injection container.

## Troubleshooting

### Compilation Errors

If you see compilation errors about unmapped properties:

1. Ensure property names match between source and destination
2. Use `[MapProperty]` for differently named properties
3. Use `[MapperIgnoreTarget]` or `[MapperIgnoreSource]` for properties that shouldn't be mapped

### Missing Mappings

If a mapping method isn't generated:

1. Ensure the class has the `[Mapper]` attribute
2. Ensure the class is `partial`
3. Ensure the method is `partial`
4. Check that the Mapperly NuGet package is installed

### Viewing Generated Code

To inspect the generated mapping code:

1. Build your project
2. Look in `obj/Debug/net8.0/generated/Riok.Mapperly/` directory
3. Or use "Go to Definition" (F12) on the mapper method in your IDE

## See Also

- [DTO Mappings](Infrastructure-Core-Mvc-Dto-Mappings.md)
- [Mapperly Documentation](https://mapperly.riok.app/docs/intro/)
- [Data Transfer Objects](https://aspnetboilerplate.com/Pages/Documents/Data-Transfer-Objects)
