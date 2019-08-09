# Changing GetPeople Method

We're changing **PersonAppService.GetPeople** method to **include**
phone numbers of people into return value.

First, we're changing **PersonListDto** to contain a list of phones:

```csharp
public class PersonListDto : FullAuditedEntityDto
{
    public string Name { get; set; }

    public string Surname { get; set; }

    public string EmailAddress { get; set; }

    public Collection<PhoneInPersonListDto> Phones { get; set; }
}

public class PhoneInPersonListDto : CreationAuditedEntityDto<long>
{
    public PhoneType Type { get; set; }

    public string Number { get; set; }
}
```

So, added also a DTO to transfer phone numbers and mapped from Phone
entity. Now, we can change GetPeople method to get Phones from database:

```csharp
public ListResultDto<PersonListDto> GetPeople(GetPeopleInput input)
{
    var persons = _personRepository
        .GetAll()
        .Include(p => p.Phones)
        .WhereIf(
            !input.Filter.IsNullOrEmpty(),
            p => p.Name.Contains(input.Filter) ||
                    p.Surname.Contains(input.Filter) ||
                    p.EmailAddress.Contains(input.Filter)
        )
        .OrderBy(p => p.Name)
        .ThenBy(p => p.Surname)
        .ToList();

    return new ListResultDto<PersonListDto>(ObjectMapper.MapTo<List<PersonListDto>>(persons));
}
```

And create mapping in CustomDtoMapper.cs:

```csharp
configuration.CreateMap<Phone, PhoneInPersonListDto>();
```

We only added **Include** extension method to the query. Rest of the
codes remains same. Furthermore, it would work without adding this, but
much slower (since it will lazy load phone numbers for every person
separately).

## Next

- [Adding AddPhone and DeletePhone Methods](Developing-Step-By-Step-Core-Adding-AddPhone-DeletePhone-Methods.md)