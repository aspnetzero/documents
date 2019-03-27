# Creating Person Application Service

An Application Service is used from client (presentation layer) to
perform operations (use cases) in the application.

Application services are located in **.Application** project and their
interfaces are located in **Application.Shared** project. We create
first application service to get people from server. We're creating an
**interface** to define the person application service (while this
interface is optional, we suggest you to create it):

```csharp
public interface IPersonAppService : IApplicationService
{
    ListResultDto<PersonListDto> GetPeople(GetPeopleInput input);
}
```

An application service method gets/returns **DTO**s. We Place them in
**Application.Shared** project. **ListResultDto** is a pre-build helper
DTO to return a list of another DTO. **GetPeopleInput** is a DTO to pass
request parameters to **GetPeople** method. So, GetPeopleIntput and
PersonListDto are defined as shown below:

```csharp
public class GetPeopleInput
{
    public string Filter { get; set; }
}

public class PersonListDto : FullAuditedEntityDto
{
    public string Name { get; set; }

    public string Surname { get; set; }

    public string EmailAddress { get; set; }
}
```

**CustomDtoMapper.cs** is used to create mapping from **Person** to
**PersonListDto**. **FullAuditedEntityDto** is inherited to
implement audit properties automatically. See [application
service](https://aspnetboilerplate.com/Pages/Documents/Application-Services)
and
[DTO](https://aspnetboilerplate.com/Pages/Documents/Data-Transfer-Objects)
documentations for more information. We are adding the following mappings.

```csharp
configuration.CreateMap<Person, PersonListDto>();
```

After defining interface, we can implement it as shown below:

```csharp
public class PersonAppService : PhoneBookDemoAppServiceBase, IPersonAppService
{
    private readonly IRepository<Person> _personRepository;

    public PersonAppService(IRepository<Person> personRepository)
    {
        _personRepository = personRepository;
    }

    public ListResultDto<PersonListDto> GetPeople(GetPeopleInput input)
    {
        var persons = _personRepository
            .GetAll()
            .WhereIf(
                !input.Filter.IsNullOrEmpty(),
                p => p.Name.Contains(input.Filter) ||
                        p.Surname.Contains(input.Filter) ||
                        p.EmailAddress.Contains(input.Filter)
            )
            .OrderBy(p => p.Name)
            .ThenBy(p => p.Surname)
            .ToList();

        return new ListResultDto<PersonListDto>(ObjectMapper.Map<List<PersonListDto>>(persons));
    }
}
```

We're injecting **person repository** (it's automatically created by
ABP) and using it to filter and get people from database.

**WhereIf** is an extension method here (defined in Abp.Linq.Extensions
namespace). It performs Where condition, only if filter is not null or
empty. **IsNullOrEmpty** is also an extension method (defined in
Abp.Extensions namespace). ABP has many similar shortcut extension
methods. **ObjectMapper.Map** method automatically converts list of
Person entities to list of PersonListDto objects with using
configurations in **CustomDtoMapper.cs** in **.Application** project.

## Connection & Transaction Management

We don't manually open database connection or start/commit transactions
manually. It's automatically done with ABP framework's Unit Of Work
system. See [UOW
documentation](https://aspnetboilerplate.com/Pages/Documents/Unit-Of-Work)
for more.

## Exception Handling

We don't handle exceptions manually (using a try-catch block). Because
ABP framework automatically handles all exceptions on the web layer and
returns appropriate error messages to the client. It then handles errors
on the client and shows needed error information to the user. See
[exception handling
document](https://aspnetboilerplate.com/Pages/Documents/Handling-Exceptions)
for more.

## Next

- [Creating Unit Tests For PersonAppService](Developing-Step-By-Step-Core-Creating-Unit-Tests-PersonAppService.md)				
