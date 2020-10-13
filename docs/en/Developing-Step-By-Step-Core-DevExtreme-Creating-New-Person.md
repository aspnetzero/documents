# Creating a New Person

Next step is to create a modal to add a new item to phone book.

## Adding CreatePerson Method to PersonAppService

We first define **CreatePerson** method in **IPersonAppService**
interface:

```csharp
Task CreatePerson(CreatePersonInput input);
```

Then we create **CreatePersonInput** DTO that defines parameters of the
method:

```csharp
public class CreatePersonInput
{
    [Required]
    [MaxLength(PersonConsts.MaxNameLength)]
    public string Name { get; set; }

    [Required]
    [MaxLength(PersonConsts.MaxSurnameLength)]
    public string Surname { get; set; }

    [EmailAddress]
    [MaxLength(PersonConsts.MaxEmailAddressLength)]
    public string EmailAddress { get; set; }
}
```

And create mapping in CustomDtoMapper.cs:

```csharp
configuration.CreateMap<CreatePersonInput, Person>();
```

All properties are decorated with **data annotation attributes**
to provide automatic
**[validation](https://aspnetboilerplate.com/Pages/Documents/Validating-Data-Transfer-Objects)**.
Notice that we use same consts defined in Person entity for
**MaxLength** properties.

Here, the implementation of CreatePerson method:

```csharp
public async Task CreatePerson(CreatePersonInput input)
{
    var person = ObjectMapper.Map<Person>(input);
    await _personRepository.InsertAsync(person);
}
```

A Person entity is created by mapping given input, then inserted to
database. We used **async/await** pattern here. All methods in ASP.NET
Zero startup project is **async**. It's advised to use async/await
wherever possible.

## Next

- [Testing CreatePerson Method](Developing-Step-By-Step-Core-DevExtreme-Testing-CreatePerson-Method.md)
