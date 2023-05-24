# Using ASP.NET Zero with DevExtreme Mvc Part 2

In this second part of the tutorial, we will continue building the phonebook application using ASP.NET Zero and DevExtreme. We will focus on creating the `PersonApplicationService`, implementing **CRUD** (Create, Read, Update, Delete) operations for the person entities. We will also cover **unit testing** the `PersonApplicationService` and using the `GetPeople` method from an Mvc component.

## Creating Person Application Service

You can skip this section if you are not interested in **automated testing**.

By writing unit test, we can test `GetPeople` method of `PersonAppService` without creating a user interface. We write unit test in `.Tests` project in the solution.

```csharp
public interface IPersonAppService : IApplicationService
{
    ListResultDto<PersonListDto> GetPeople(GetPeopleInput input);
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

# Testing PersonAppService From Browser Console

Now, lets run and **login** to the application again, open Chrome Developer Console (or similar tools in other browsers) and write the following command:

<img src="/Images/Blog/chrome-console-get-people.png" alt="Google Chrome Console AJAX" class="img-thumbnail" width="497" height="104" />

This command performs an **AJAX** call to `GetPeople` method of `PersonAppService`. We can see the request if we open **Network** tab:

<img src="/Images/Blog/chrome-console-get-people-network.png" alt="Google Chrome Console Get People" class="img-thumbnail" width="720" height="148" />

As we see, an AJAX request made and people are got successfully.

So, how this happen? How we could make a call to a C\# class method (notice that it's not an MVC Controller, just a plain C\# class) from javascript like calling a javascript method? This is provided by ASP.NET Boilerplate. See [AspNet Core](https://aspnetboilerplate.com/Pages/Documents/AspNet-Core#application-services-as-controllers) documentation for more information. You can always call application services from console to debug or see returned JSON structure.

We can also check Audit Logs to see the request. Open `AbpAuditLogs` table in `PhoneBook` database to see the call information:

<img src="/Images/Blog/audit-log-table-getpeople.png" alt="Audit Logs table" class="img-thumbnail" width="921" height="71" />

There are some other fields not shown here. So, we see that User with Id=2 called `GetPeople` method of the `PersonAppService` in recorded time with the shown parameters and it's executed in 134 ms.

# Using `GetPeople` Method From MVC Controller

It's time to open `PhoneBookController` and get people to show on the view:

```csharp
using Abp.Web.Models;
using DevExtreme.AspNet.Data;
using DevExtreme.AspNet.Mvc;

[Area("App")]
public class PhoneBookController : PhoneBookDemoControllerBase
{
    private readonly IPersonAppService _personAppService;

    public PhoneBookController(IPersonAppService personAppService)
    {
        _personAppService = personAppService;
    }

    public ActionResult Index()
    {    
        return View();
    }
    
    [WrapResult(false,false)]
    public object LoadPeople(GetPeopleInput input, DataSourceLoadOptions loadOptions)
    {            
        var output = _personAppService.GetPeople(input);
        return DataSourceLoader.Load(output.Items, loadOptions);
    }
}
```

We inject `IPersonAppService` and call its `GetPeople` method (which is created and tested before) to get list of people. Then we used `DataSourceLoader` Load to return data that DevExtreme's DataGrid wants.

## Application Services and ViewModels

We created an **Application Service** (`PersonAppService`) and used it from the **Controller**. Instead, we could access **Repository** directly from Controller and completely discard the application service. ASP.NET Zero does not enforce any architecture here. In ASP.NET Zero, we
use the application layer (application services and DTOs). Therefore, we implemented it **independent** from ASP.NET MVC. This makes application
layer re-usable from different presentation layers. But if you will only develop ASP.NET MVC, you can implement application logic inside controllers and access to the repositories from controllers. This may simplify your architecture and development model.

If you decide to develop application services and use them in controllers then you can use application service's **output** as your view model. We did not prefer it and wrapped output by a dedicated ViewModel (`IndexViewModel` here) since we think that we may add some **additional properties/methods** for our view model. Again, it's your choice of implementation.

## Rendering People In MVC View

We show people on the page is most basic form. See the changed view below:

```html
@using Acme.PhoneBookDemo.Web.Areas.App.Startup
@using DevExtreme.AspNet.Mvc
@{
    ViewBag.CurrentPageName = AppPageNames.Tenant.PhoneBook;
}
<div class="content d-flex flex-column flex-column-fluid" id="kt_content">
    <abp-page-subheader title="@L("PhoneBook")" description="@L("PhoneBookInfo")"></abp-page-subheader>

    <div class="@(await GetContainerClass())">
        <div class="col-12">
            <div class="card card-custom gutter-b">
                <div class="card-body">
                    @(Html.DevExtreme()
                        .DataGrid()              
                        .DataSource(d => d.Mvc()
                            .Controller("PhoneBook")
                            .LoadAction("LoadPeople")
                            .Key("id")
                        )
                        .Columns(columns =>
                        {
                            columns.Add()
                                .DataField("name")
                                .Caption(L("Name"));
                            
                            columns.Add()
                                .DataField("surname")
                                .Caption(L("Surname"));
                            
                            columns.Add()
                                .DataField("emailAddress")
                                .Caption(L("EmailAddress"));                       
                        })
                    )
                </div>
            </div>
        </div>
    </div>
</div>
```

We successfully retrieved list of people from database to the page. See the result:

<img src="/Images/Blog/phonebook-devextreme-people-view-1.png" alt="Phonebook peoples" class="img-thumbnail"/>

## Conclusion

In this blog post, we've seen how to create a new **application service** and use it from Mvc. We've also seen how to write **unit tests** for the application service. 

### Summary

* We've created a new application service `PersonAppService` and added a new method `GetPeople` to it.

* We've added a new unit test to test `GetPeople` method.

* We've used `GetPeople` method from Mvc view.

* We've add a `PhoneBookComponent.html` to show people in a table.

### What's Next?

In our upcoming blog post, we will focus on the implementation of the `CreatePerson` method. We'll create a new page dedicated to creating new people and ensure the functionality is thoroughly tested through unit tests. Additionally, we'll explore the capabilities of **DevExtreme** controls within the Mvc framework.

See the [next part](https://aspnetzero.com/blog/using-asp.net-zero-with-devextreme-mvc-part-3) of this tutorial.