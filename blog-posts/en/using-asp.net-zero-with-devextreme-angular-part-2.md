# Using ASP.NET Zero with DevExtreme Angular Part 2

In this second part of the tutorial, we will continue building the phonebook application using ASP.NET Zero and DevExtreme. We will focus on creating the `PersonApplicationService`, implementing **CRUD** (Create, Read, Update, Delete) operations for the person entities. We will also cover **unit testing** the `PersonApplicationService` and using the `GetPeople` method from an Angular component.

## Creating Person Application Service

Application service interface and DTOs are located in `.Application.Shared` project. We are creating an application service to get people from the server. So, we're first creating an **interface** to define the person application service (while this interface is optional, we suggest you to create it):

```csharp
using Abp.Application.Services;
using Abp.Application.Services.Dto;

namespace Acme.PhoneBookDemo.PhoneBook
{
    public interface IPersonAppService : IApplicationService
    {
        ListResultDto<PersonListDto> GetPeople(GetPeopleInput input);
    }
}
```

An application service method gets/returns DTOs. `ListResultDto` is a pre-build helper DTO to return a list of another DTO. `GetPeopleInput` is a DTO to pass request parameters to `GetPeople` method. So, `GetPeopleIntput` and `PersonListDto` are defined as shown below:

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

`CustomDtoMapper.cs` is used to create mapping from `Person` to `PersonListDto`. `FullAuditedEntityDto` is inherited to implement audit properties automatically. See [application service](https://aspnetboilerplate.com/Pages/Documents/Application-Services) and [DTO](https://aspnetboilerplate.com/Pages/Documents/Data-Transfer-Objects) documentations for more information. We are adding the following mappings.

```csharp
...
// PhoneBook (we will comment out other lines when the new DTOs are added)
configuration.CreateMap<Person, PersonListDto>();
//configuration.CreateMap<AddPhoneInput, Phone>();
//configuration.CreateMap<CreatePersonInput, Person>();
//configuration.CreateMap<Person, GetPersonForEditOutput>();
//configuration.CreateMap<Phone, PhoneInPersonListDto>();
```

After defining interface, we can implement it as shown below: (in `.Application` project)

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

We're injecting **person repository** (it's automatically created by ABP) and using it to filter and get people from database. 

`WhereIf` is an extension method here (defined in `Abp.Linq.Extensions` namespace). It performs Where condition, only if filter is not null or empty. `IsNullOrEmpty` is also an extension method (defined in `Abp.Extensions` namespace). ABP has many similar shortcut extension methods. `ObjectMapper.Map` method automatically converts list of `Person` entities to list of `PersonListDto` objects with using configurations in `CustomDtoMapper.cs` in `.Application` project.

**Note:** We don't manually open database connection or start/commit transactions manually. It's automatically done with ABP framework's Unit Of Work system. See [UOW documentation](https://aspnetboilerplate.com/Pages/Documents/Unit-Of-Work) for more.

**Note:** We don't handle exceptions manually (using a try-catch block). Because ABP framework automatically handles all exceptions on the web layer and returns appropriate error messages to the client. It then handles errors on the client and shows needed error information to the user. See [exception handling document](https://aspnetboilerplate.com/Pages/Documents/Handling-Exceptions) for more.

## Creating Unit Tests for Person Application Service

You can skip this section if you are not interested in **automated testing**.

By writing unit test, we can test `GetPeople` method of `PersonAppService` without creating a user interface. We write unit test in `.Tests` project in the solution.

## MultiTenancy In Tests

Since we disabled multitenancy, we should disable it for unit tests too. Open `PhoneBookDemoConsts` class in the `Acme.PhoneBook.Core` project and set `MultiTenancyEnabled` to false. After a rebuild and run unit tests, you will see that some tests are skipped (those are related to multitenancy).

```csharp
public class PersonAppService_Tests : AppTestBase
{
    private readonly IPersonAppService _personAppService;

    public PersonAppService_Tests()
    {
        _personAppService = Resolve<IPersonAppService>();
    }

    [Fact]
    public void Should_Get_All_People_Without_Any_Filter()
    {
        //Act
        var persons = _personAppService.GetPeople(new GetPeopleInput());

        //Assert
        persons.Items.Count.ShouldBe(2);
    }
}
```

We derived test class from `AppTestBase`. `AppTestBase` class initializes all system, creates an in-memory fake database, seeds initial data (that we created before) to database and logins to application as admin. So, this is actually an **integration test** since it tests all server-side code from entity framework mapping to application services, validation and authorization.

In constructor, we get (resolve) an `IPersonAppService` from **dependency injection** container. It creates the `PersonAppService` class with all dependencies. Then we can use it in test methods.

Since we're using [xUnit](http://xunit.github.io/), we add `Fact` attribute to each test method. In the test method, we called `GetPeople` method and checked if there are **two people** in the returned list as we know that there were 2 people in **initial** database.

Let's run the **all unit tests** in Test Explorer and see if it works:

<img src="/Images/Blog/phone-book-unit-test-success-2.png" alt="xUnit unit test success" class="img-thumbnail" />

As you see, it worked **successfully**. Now, we know that `PersonAppService` works properly without any filter. Let's add a new unit test to get filtered people:

```csharp
[Fact]
public void Should_Get_People_With_Filter()
{
    //Act
    var persons = _personAppService.GetPeople(
        new GetPeopleInput
        {
            Filter = "adams"
        });

    //Assert
    persons.Items.Count.ShouldBe(1);
    persons.Items[0].Name.ShouldBe("Douglas");
    persons.Items[0].Surname.ShouldBe("Adams");
}
```

Again, since we know initial database, we can check returned results easily. Here, initial test data is important. When we change initial data, our test may fail even if our services are correct. So, it's better to write unit tests independent of initial data as much as possible. We could check incoming data to see if every people contains "adams" in his/her name, surname or email. Thus, if we add new people to initial data, our tests remain working.

There are many techniques on unit testing, I kept it simple here. But ASP.NET Zero template makes very easy to write unit and integration tests by base classes and pre-build test codes. 

## Using `GetPeople` Method From Angular Component

Now, we can switch to the client side and use `GetPeople` method to show a list of people on the UI.

### Service Proxy Generation

First, run (prefer **Ctrl+F5** to be faster) the server side application (`.Web.Host` project). Then run `nswag/refresh.bat` file on the client side to re-generate service proxy classes (they are used to call server side service methods).

Since we added a new service, we should add it to `src/shared/service-proxies/service-proxy.module.ts`. Just open it and add `ApiServiceProxies.PersonServiceProxy` to the providers array. This step is only required when we add a new service. If we change an existing service, it's not needed.

### PhoneBookComponent Typescript Class

Change `phonebook.component.ts` as like below:

```typescript
import {Component, Injector} from '@angular/core';
import {AppComponentBase} from '@shared/common/app-component-base';
import {appModuleAnimation} from '@shared/animations/routerTransition';
import {PersonServiceProxy} from "@shared/service-proxies/service-proxies";
import CustomStore from "@node_modules/devextreme/data/custom_store";

@Component({
    templateUrl: './phonebook.component.html',
    animations: [appModuleAnimation()]
})

export class PhoneBookComponent extends AppComponentBase {

    dataSource: any;
    refreshMode: string;

    constructor(
        injector: Injector,
        private _personService: PersonServiceProxy
    ) {
        super(injector);
        this.getData();
    }

    getData() {
        this.refreshMode = "full";

        this.dataSource = new CustomStore({
            key: "id",
            load: (loadOptions) => {
                return this._personService.getPeople("").toPromise();
            }
        });
    }
}

```

We inject `PersonServiceProxy`, call its `getPeople` method and `subscribe` to get the result. We do this in `ngOnInit` function (defined in Angular's `OnInit` interface). Assigned returned items to the `people` class member.

### Rendering People In Angular View

Now, we can use this people member from the view.

*phonebook.component.html*

```html
<div [@routerTransition]>
    <div class="content d-flex flex-column flex-column-fluid">
        <sub-header [title]="'PhoneBook' | localize">
        </sub-header>

        <div [class]="containerClass">
            <div class="card card-custom">
                <div class="card-body">
                    <dx-data-grid
                        id="phonebookgrid"
                        [dataSource]="dataSource"
                        [repaintChangesOnly]="true"
                        [showBorders]="true">

                        <dxo-scrolling mode="virtual"></dxo-scrolling>

                        <dxi-column dataField="name" caption="{{'Name' | localize}}"></dxi-column>
                        <dxi-column dataField="surname" caption="{{'Surname' | localize}}"></dxi-column>
                        <dxi-column dataField="emailAddress" caption="{{'EmailAddress' | localize}}"></dxi-column>
                    </dx-data-grid>
                </div>
            </div>
        </div>
    </div>
</div>

```

We successfully retrieved list of people from database to the page. See the result:

<img src="/Images/Blog/phonebook-people-view-angular-1.png" alt="Phonebook peoples" class="img-thumbnail" />

## Conclusion

In this blog post, we've seen how to create a new **application service** and use it from Angular. We've also seen how to write **unit tests** for the application service. 

### Summary

* We've created a new application service `PersonAppService` and added a new method `GetPeople` to it.

* We've added a new unit test to test `GetPeople` method.

* We've used `GetPeople` method from Angular component `PhoneBookComponent.ts`.

* We've add a `PhoneBookComponent.html` to show people in a table.

### What's Next?

In our upcoming blog post, we will focus on the implementation of the `CreatePerson` method. We'll create a new page dedicated to creating new people and ensure the functionality is thoroughly tested through unit tests. Additionally, we'll explore the capabilities of **DevExtreme** controls within the Angular framework.

https://aspnetzero.com/blog/using-asp.net-zero-with-devextreme-angular-part-3

