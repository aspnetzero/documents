# Creating Person Application Service

An Application Service is used from client (presentation layer) to perform operations (use cases) of the application.

Application service interface and DTOs are located in **.Application.Shared** project. We are creating an application service to get people from the server. So, we're first creating an **interface** to define the person application service (while this interface is optional, we suggest you to create it):

```csharp
using Abp.Application.Services;
using Abp.Application.Services.Dto;

namespace Acme.PhoneBookDemo.PhoneBook;

public interface IPersonAppService : IApplicationService
{
    ListResultDto<PersonListDto> GetPeople(GetPeopleInput input);
}
```

An application service method gets/returns **DTO**s. **ListResultDto** is a pre-build helper DTO to return a list of another DTO. **GetPeopleInput** is a DTO to pass request parameters to **GetPeople** method. So, GetPeopleIntput and PersonListDto are defined as shown below:

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

We need to create a mapper to convert **Person** to **PersonListDto**. **FullAuditedEntityDto** is inherited to implement audit properties automatically. See [application service](https://aspnetboilerplate.com/Pages/Documents/Application-Services) and [DTO](https://aspnetboilerplate.com/Pages/Documents/Data-Transfer-Objects) documentations for more information.

Create a new file `PersonToPersonListDtoMapper.cs` in the **.Application** project:

```csharp
using Riok.Mapperly.Abstractions;

namespace Acme.PhoneBookDemo.PhoneBook.Mapper;

[Mapper]
public partial class PersonToPersonListDtoMapper
{
    public partial PersonListDto Map(Person person);
    public partial List<PersonListDto> Map(List<Person> persons);
}
```

ASP.NET Zero automatically discovers and registers this mapper class.

After defining interface, we can implement it as shown below: (in **.Application** project)

```csharp
using Abp.Application.Services.Dto;
using Abp.Collections.Extensions;
using Abp.Domain.Repositories;
using System.Collections.Generic;
using System.Linq;

namespace Acme.PhoneBookDemo.PhoneBook;

public class PersonAppService : PhoneBookDemoAppServiceBase, IPersonAppService
{
    private readonly IRepository<Person> _personRepository;

    public PersonAppService(IRepository<Person> personRepository)
    {
        _personRepository = personRepository;
    }

    public ListResultDto<PersonListDto> GetPeople(GetPeopleInput input)
    {
        var people = _personRepository
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

        return new ListResultDto<PersonListDto>(ObjectMapper.Map<List<PersonListDto>>(people));
    }
}
```

We're injecting **person repository** (it's automatically created by ABP).

**WhereIf** is an extension method here (defined in Abp.Linq.Extensions namespace). It performs Where condition, only if filter is not null or empty. **IsNullOrEmpty** is also an extension method (defined in Abp.Extensions namespace). ABP has many similar shortcut extension methods. **ObjectMapper.Map** automatically converts list of Person entities to list of PersonListDto objects using the Mapperly-generated mapping code internally.

### Connection & Transaction Management

We don't manually open database connection or start/commit transactions manually. It's automatically done with ABP framework's Unit Of Work system. See [UOW documentation](https://aspnetboilerplate.com/Pages/Documents/Unit-Of-Work) for more.

### Exception Handling

We don't handle exceptions manually (using a try-catch block). Because ABP framework automatically handles all exceptions on the web layer and returns appropriate error messages to the client. It then handles errors on the client and shows needed error information to the user. See [exception handling document](https://aspnetboilerplate.com/Pages/Documents/Handling-Exceptions) for more.

## Next

- [Using GetPeople Method From Angular Component](Developing-Step-By-Step-Angular-Using-GetPeople-Method-from-Angular)