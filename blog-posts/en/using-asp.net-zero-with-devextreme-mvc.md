# Using ASP.NET Zero with DevExtreme MVC

In this document, we will create a sample **phonebook application** based on ASP.NET Zero (ASP.NET Core version) step by step. After all steps, we will have a multi-tenant, localized, authorized, configurable,

First of all you should follow that directions and implement DevExtreme to your existing project.

Non visual studio project: https://docs.devexpress.com/AspNetCore/401027/devextreme-based-controls/get-started/configure-a-non-visual-studio-project?v=19.1

Visual studio project: https://docs.devexpress.com/AspNetCore/401026/devextreme-based-controls/get-started/configure-a-visual-studio-project?v=19.1

Then, go to `bundles.json` file of  `*.Mvc`  project. 

Find `app-layout-libs.min.js` and remove `node_modules/jquery/dist/jquery.js` from it.

## Creating & Running the Project

We're creating and downloading the solution named "**Acme.PhoneBookDemo**" as described in [Getting
Started](Getting-Started-Core) document. After opening solution in Visual Studio, we see an NLayered solution that consists of eight projects:

<img src="/Images/Blog/solution-overall-core-5.png" alt="Solution Overall" class="img-thumbnail" />

Also, run database migrations, create the database and login to the application as described in [Getting Started](Getting-Started-Core) document. After all completed and logged in to the application, we see a dashboard as shown below:

<img src="/Images/Blog/default-dashboard3.png" alt="Dashboard" class="img-thumbnail" />

Logout from the application for now. We will make our application **single-tenant** (we will convert it to multi-tenant later). So, we open PhoneBookDemoConsts class in the Acme.PhoneBookDemo.Core project
and disable multi-tenancy as shown below:

```csharp
public class PhoneBookDemoConsts
{
    public const string LocalizationSourceName = "PhoneBookDemo";

    public const string ConnectionStringName = "Default";

    public const bool MultiTenancyEnabled = false;

    public const int PaymentCacheDurationInMinutes = 30;
}
```

# Adding a New Menu Item

Let's begin from UI and create a new page named "**Phone book**".

## Defining a menu item

**AppNavigationProvider** class defines menus in the application. When
we change this class, menus are automatically changed. Open this class
and create new menu item as shown below (You can add it right after the
dashboard menu item).

```csharp
.AddItem(new MenuItemDefinition(
    AppPageNames.Tenant.PhoneBook,
    L("PhoneBook"),
    url: "App/PhoneBook",
    icon: "glyphicon glyphicon-book"
    )
)
```

Every menu item must have a **unique name** to identify this menu item.
Menu names are defined in AppPageNames class as constants. We add a new
constant: "**PhoneBook**".


## Localizing Menu Item Display Name

A menu item should also have a **localizable shown name**. It's used to
display menu item on the page. **L("PhoneBook")** is the localized name of
our new menu. **L** method is a helper method gets a localization key
and simply returns a **LocalizableString** object (see
AppNavigationProvider class).

Localization strings are defined in **XML** files in **.Core** project
as shown below:

<img src="/Images/Blog/localization-files-5.png" alt="Localization files" class="img-thumbnail" />

Open PhoneBook.xml (the **default**, **English** localization
dictionary) and add the following line:

```xml
<text name="PhoneBook">Phone book</text>
```

If we don't define "PhoneBook"s value for other localization
dictionaries, default value is shown in all languages. We can define it
also for Turkish in PhoneBook-tr.xml file:

```xml
<text name="PhoneBook">Telefon Rehberi</text>
```

## Other menu item properties

**url** can be a URL (it's URL of an **MVC Action** here) that will be
redirected when we click the menu item.

Lastly, **icon** is the shown menu icon for new menu item. It can be a
**css** class. We can use Glyphicon, Font-Awesome or another css font
library here.

See [navigation
document](https://aspnetboilerplate.com/Pages/Documents/Navigation) for
more information on menu definitions.

# Creating the Page

After creating the menu item, we can create an empty page.

## Controller

Creating the **PhoneBookController** under **Areas/App/Controllers**
folder in the Web project:

```csharp
[Area("App")]
public class PhoneBookController : PhoneBookDemoControllerBase
{
    public ActionResult Index()
    {
        return View();
    }
}
```

We inherited from **PhoneBookDemoControllerBase** (will be
*YourProjectName*ControllerBase for your projects) instead of MVC's
standard Controller class. While it will work if we derive from the
standard Controller, PhoneBookDemoControllerBase provides very useful
base properties and methods. So, always inherit from this class unless
it has a disadvantage for your case.

## View

Creating an empty view, **Index.cshtml** under
**Areas/App/Views/PhoneBook** folder:

```html
@using Acme.PhoneBookDemo.Web.Areas.App.Startup
@{
    ViewBag.CurrentPageName = AppPageNames.Tenant.PhoneBook;
}

<div class="content d-flex flex-column flex-column-fluid">
    <abp-page-subheader title="@L("PhoneBook")" description="@L("PhoneBookInfo")"></abp-page-subheader>
    
    <div class="@(await GetContainerClass()">          
        <div class="col-12">
            <div class="card card-custom gutter-b">
                <div class="card-body">
                    <p>PHONE BOOK CONTENT COMES HERE!</p>
                </div>
            </div>
        </div>          
    </div>
</div>
```

We set ViewBag.CurrentPageName to the current page's name to
automatically highlight the related menu item when this page is active.
Now, it's time to run application and see the new phone book page:

<img src="/Images/Blog/phonebook-empty-mpa2.png" alt="Phone book empty screen" class="img-thumbnail" />

Menu item display name and page title are localized. Try to change UI
language to see difference.

# Creating Person Entity

We define entities in **.Core** (domain) project. We can define a
**Person** entity (mapped to **PbPersons** table in database) to
represent a person in phone book as shown below:

```csharp
[Table("PbPersons")]
public class Person : FullAuditedEntity
{
    [Required]
    [MaxLength(PersonConsts.MaxNameLength)]
    public virtual string Name { get; set; }

    [Required]
    [MaxLength(PersonConsts.MaxSurnameLength)]
    public virtual string Surname { get; set; }

    [MaxLength(PersonConsts.MaxEmailAddressLength)]
    public virtual string EmailAddress { get; set; }
}
```

Person's **primary key** type is **int** (as default). It inherits
**FullAuditedEntity** that contains **creation**, **modification** and
**deletion** audit properties. It's also **soft-delete**. When we delete
a person, it's not deleted by database but marked as deleted (see [entity](https://aspnetboilerplate.com/Pages/Documents/Entities) and [data filters](https://aspnetboilerplate.com/Pages/Documents/Data-Filters) documentations for more information). We created **PersonConsts** in **Core.Shared** project for **MaxLength** properties. This is a good practice since we will use same values later.

```csharp
public class PersonConsts
{
    public const int MaxNameLength = 32;
    public const int MaxSurnameLength = 32;
    public const int MaxEmailAddressLength = 255;
}
```

We add a DbSet property for Person entity to **PhoneBookDemoDbContext** class defined in **.EntityFrameworkCore** project.

```csharp
public class PhoneBookDemoDbContext : AbpZeroDbContext<Tenant, Role, User, PhoneBookDemoDbContext>
{
    public virtual DbSet<Person> Persons { get; set; }

    //...other entities

    public PhoneBookDemoDbContext()
        : base("Default")
    {

    }

    //...other codes
}
```

# Database Migrations for Person

We use **Entity Framework Code-First migrations** to migrate database
schema. Since we added **Person entity**, our DbContext model is
changed. So, we should create a **new migration** to create the new
table in the database.

Open **Package Manager Console**, run the **Add-Migration
"Added\_Persons\_Table"** command as shown below:

<img src="/Images/Blog/phonebook-migrations-core-3.png" alt="Entity Framework Code First Migration" class="img-thumbnail" />

This command will add a **migration class** named
"**Added\_Persons\_Table**" as shown below:

```csharp
public partial class Added_Persons_Table : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "PbPersons",
            columns: table => new
            {
                Id = table.Column(nullable: false)
                    .Annotation("SqlServer:ValueGenerationStrategy", SqlServerValueGenerationStrategy.IdentityColumn),
                CreationTime = table.Column(nullable: false),
                CreatorUserId = table.Column(nullable: true),
                DeleterUserId = table.Column(nullable: true),
                DeletionTime = table.Column(nullable: true),
                EmailAddress = table.Column(maxLength: 255, nullable: true),
                IsDeleted = table.Column(nullable: false),
                LastModificationTime = table.Column(nullable: true),
                LastModifierUserId = table.Column(nullable: true),
                Name = table.Column(maxLength: 32, nullable: false),
                Surname = table.Column(maxLength: 32, nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_PbPersons", x => x.Id);
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(
            name: "PbPersons");
    }
}
```

We don't have to know so much about format and rules of this file. But,
it's suggested to have a basic understanding of migrations. In the same
Package Manager Console, we write **Update-Database** command in order
to apply the new migration to database. After updating, we can see that
**PbPersons table** is added to database.

<img src="/Images/Blog/phonebook-tables-mpa.png" alt="Phonebook tables" class="img-thumbnail" width="192" height="370" />

But this new table is empty. In ASP.NET Zero, there are some classes to
fill initial data for users and settings:

<img src="/Images/Blog/aspnet-core-ef-seed-1.png" alt="Seed folders" class="img-thumbnail" />

So, we can add a separated class to fill some people to database as
shown below:

```csharp
namespace Acme.PhoneBookDemo.Migrations.Seed.Host
{
    public class InitialPeopleCreator
    {
        private readonly PhoneBookDemoDbContext _context;

        public InitialPeopleCreator(PhoneBookDemoDbContext context)
        {
            _context = context;
        }

        public void Create()
        {
            var douglas = _context.Persons.FirstOrDefault(p => p.EmailAddress == "douglas.adams@fortytwo.com");
            if (douglas == null)
            {
                _context.Persons.Add(
                    new Person
                    {
                        Name = "Douglas",
                        Surname = "Adams",
                        EmailAddress = "douglas.adams@fortytwo.com"
                    });
            }

            var asimov = _context.Persons.FirstOrDefault(p => p.EmailAddress == "isaac.asimov@foundation.org");
            if (asimov == null)
            {
                _context.Persons.Add(
                    new Person
                    {
                        Name = "Isaac",
                        Surname = "Asimov",
                        EmailAddress = "isaac.asimov@foundation.org"
                    });
            }
        }
    }
}
```

These type of default data is good since we can also use these data in
**unit tests**. Surely, we should be careful about seed data since this
code will always be executed in each **PostInitialize** of your
PhoneBookEntityFrameworkCoreModule. This class (InitialPeopleCreator) is
created and called in **InitialHostDbBuilder** class. This is not so
important, just for a good code organization (see source codes).

```csharp
public class InitialHostDbBuilder
{
    //existing codes...

    public void Create()
    {
        //existing codes...
        new InitialPeopleCreator(_context).Create();

        _context.SaveChanges();
    }
}
```

We run our project again, it runs seed and adds two people to PbPersons
table:

<img src="/Images/Blog/phonebook-persons-table-initial-data.png" alt="Persons initial data" class="img-thumbnail" width="720" height="50" />

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

# Creating Unit Tests For PersonAppService

You can skip this section if you don't interest in **automated
testing**.

By writing unit test, we can test **PersonAppService.GetPeople** method
without creating a user interface that calls it and shows people on the
screen.

We write unit test in .**Tests** project in the solution. Let's create
first test to verify getting people without any filter:

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

We derived test class from **AppTestBase**. AppTestBase class
initializes all system, creates an in-memory fake database, seeds
initial data (that we created before) to database and logins to
application as admin. So, this is actually an **integration test** since
it tests all server-side codes from entity framework mapping to
application services, validation and authorization.

In constructor, we get (resolve) an **IPersonAppService** from
**dependency injection** container. It creates the **PersonAppService**
class with all dependencies. Then we can use it in test methods.

Since we're using [xUnit](http://xunit.github.io/), we add **Fact**
attribute to each test method. In the test method, we called
**GetPeople** method and checked if there are **two people** in the
returned list as we know that there were 2 people in **initial**
database.

Let's run the **all unit tests** in Test Explorer and see if it works:

<img src="/Images/Blog/phone-book-unit-test-success-1.png" alt="xUnit unit test success" class="img-thumbnail" width="750" height="40" />

As you see, it worked **successfully**. Now, we know that
PersonAppService works properly without any filter. Let's add a new unit
test to get filtered people:

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

Again, since we know initial database, we can check returned results
easily. Here, initial test data is important. When we change initial
data, our test may fail even if our services are correct. So, it's
better to write unit tests independent of initial data as much as
possible. We could check incoming data to see if every people contains
"adams" in his/her name, surname or email. Thus, if we add new people to
initial data, our tests remain working.

There are many techniques on unit testing, I kept it simple here. But
ASP.NET Zero template makes very easy to write unit and integration
tests by base classes and pre-build test codes.

# Testing PersonAppService From Browser Console

Now, lets run and **login** to the application again, open Chrome
Developer Console (or similar tools in other browsers) and write the
following command:

<img src="/Images/Blog/chrome-console-get-people.png" alt="Google Chrome Console AJAX" class="img-thumbnail" width="497" height="104" />

This command performs an **AJAX** call to PersonAppService.**GetPeople**
method. We can see the request if we open **Network** tab:

<img src="/Images/Blog/chrome-console-get-people-network.png" alt="Google Chrome Console Get People" class="img-thumbnail" width="720" height="148" />

As we see, an AJAX request made and people are got successfully.

So, how this happen? How we could make a call to a C\# class method
(notice that it's not an MVC Controller, just a plain C\# class) from
javascript like calling a javascript method? This is provided by ASP.NET
Boilerplate. See [AspNet
Core](https://aspnetboilerplate.com/Pages/Documents/AspNet-Core#application-services-as-controllers)
documentation for more information. You can always call application
services from console to debug or see returned JSON structure.

We can also check Audit Logs to see the request. Open **AbpAuditLogs**
table in PhoneBook database to see the call information:

<img src="/Images/Blog/audit-log-table-getpeople.png" alt="Audit Logs table" class="img-thumbnail" width="921" height="71" />

There are some other fields not shown here. So, we see that User with
Id=2 called GetPeople method of the PersonAppService in recorded time
with the shown parameters and it's executed in 134 ms.

# Using GetPeople Method From MVC Controller

It's time to open **PhoneBookController** and get people to show on the
view:

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

We inject **IPersonAppService** and call its **GetPeople** method
(which is created and tested before) to get list of people. Then we
used DataSourceLoader Load to return data that DevExtreme's DataGrid wants.

## Application Services and ViewModels

We created an **Application Service** (PersonAppService) and used it
from the **Controller**. Instead, we could access **Repository**
directly from Controller and completely discard the application service.
ASP.NET Zero does not enforce any architecture here. In ASP.NET Zero, we
use the application layer (application services and DTOs). Therefore, we
implemented it **independent** from ASP.NET MVC. This makes application
layer re-usable from different presentation layers. But if you will only
develop ASP.NET MVC, you can implement application logic inside
controllers and access to the repositories from controllers. This may
simplify your architecture and development model.

If you decide to develop application services and use them in
controllers then you can use application service's **output** as your
view model. We did not prefer it and wrapped output by a dedicated
**ViewModel** (IndexViewModel here) since we think that we may add some
**additional properties/methods** for our view model. Again, it's your
choice of implementation.

# Rendering People In MVC View

We show people on the page is most basic form. See the changed view
below:

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

See the result:

<img src="/Images/Blog/phonebook-devextreme-people-view-1.png" alt="Phonebook peoples" class="img-thumbnail"/>

We successfully retrieved list of people from database to the page.

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

# Testing CreatePerson Method

You can skip this section if you don't interest in **automated
testing**.

We can create a unit test method to test CreatePerson method as shown
below:

```csharp
[Fact]
public async Task Should_Create_Person_With_Valid_Arguments()
{
    //Act
    await _personAppService.CreatePerson(
        new CreatePersonInput
        {
            Name = "John",
            Surname = "Nash",
            EmailAddress = "john.nash@abeautifulmind.com"
        });

    //Assert
    UsingDbContext(
        context =>
        {
            var john = context.Persons.FirstOrDefault(p => p.EmailAddress == "john.nash@abeautifulmind.com");
            john.ShouldNotBe(null);
            john.Name.ShouldBe("John");
        });
}
```

Test method also written using **async/await** pattern since calling
method is async. We called CreatePerson method, then checked if given
person is in the database. **UsingDbContext** method is a helper method
of **AppTestBase** class (which we inherited this unit test class from).
It's used to easily get a reference to DbContext and use it directly to
perform database operations.

This method successfully works since all required fields are supplied.
Let's try to create a test for **invalid arguments**:

```csharp
[Fact]
public async Task Should_Not_Create_Person_With_Invalid_Arguments()
{
    //Act and Assert
    await Assert.ThrowsAsync<AbpValidationException>(
        async () =>
                {
                    await _personAppService.CreatePerson(
                        new CreatePersonInput
                        {
                            Name = "John"
                        });
                });
}
```

We did not set **Surname** property of CreatePersonInput despite it being
**required**. So, it throws **AbpValidationException** automatically.
Also, we can not send null to CreatePerson method since validation
system also checks it. This test calls CreatePerson with invalid
arguments and asserts that it throws AbpValidationException. See
[validation
document](https://aspnetboilerplate.com/Pages/Documents/Validating-Data-Transfer-Objects)
for more information.

# Creating New Person

Create a controller action to create a person.

(PhoneBookController.cs): 

```cs
[Area("App")]
public class PhoneBookController : PhoneBookDemoControllerBase
{
    private readonly IPersonAppService _personAppService;

    public PhoneBookController(IPersonAppService personAppService)
    {
        _personAppService = personAppService;
    }
    
    //...
    public async Task<IActionResult> CreatePerson(string values)
    {
        var input = new CreatePersonInput();
        JsonConvert.PopulateObject(values, input);

        if(!TryValidateModel(input))
            return BadRequest(ModelState.GetFullErrorMessage());

        await _personAppService.CreatePerson(input);
        return Ok();
    }
}
```

Then Update **Index.cshml** as seen below:

(Index.cshtml)

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
                        .Editing(editing => {
                            editing.Mode(GridEditMode.Row);
                            editing.AllowAdding(true);
                        })
                        .DataSource(d => d.Mvc()
                            .Controller("PhoneBook")
                            .LoadAction("LoadPeople")
                            .InsertAction("CreatePerson")
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

<img src="/Images/Blog/phonebook-devextreme-create-person.png" alt="Phonebook peoples" class="img-thumbnail"/>

## Deleting a Person

### Application Service

First, adding a new method definition to **IPersonAppService** interface
as always:

```csharp
Task DeletePerson(EntityDto input);
```

**EntityDto** is a shortcut of ABP if we only get an id value.
Implementation (in **PersonAppService**) is very simple:

```csharp
[AbpAuthorize(AppPermissions.Pages_Tenant_PhoneBook_DeletePerson)]
public async Task DeletePerson(EntityDto input)
{
    await _personRepository.DeleteAsync(input.Id);
}
```

We also **authorized** deleting a person as did before for creating a person.

We also need to define **Pages\_Tenant\_PhoneBook\_DeletePerson** constant in AppPermissions and define related permission in **AppAuthorizationProvider**.

### Controller

```csharp
[Area("App")]
public class PhoneBookController : PhoneBookDemoControllerBase
{
    private readonly IPersonAppService _personAppService;

    public PhoneBookController(IPersonAppService personAppService)
    {
        _personAppService = personAppService;
    }
    
    //...
    [HttpDelete]
    public async Task DeletePerson(int key)
    {
        await _personAppService.DeletePerson(new EntityDto(key));
    }
}
```

### View

We're changing **index.cshtml** view to add a button;

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
                        .Editing(editing => {
                            editing.Mode(GridEditMode.Row);
                            editing.AllowAdding(true);
                    		editing.AllowDeleting(true);
                        })
                        .DataSource(d => d.Mvc()
                            .Controller("PhoneBook")
                            .LoadAction("LoadPeople")
                            .InsertAction("CreatePerson")
                    		.DeleteAction("DeletePerson")
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

It first shows a confirmation message when we click the delete button:

<img src="/Images/Blog/phonebook-devextreme-delete-person.png" alt="Confirmation message" class="img-thumbnail" />

<img src="/Images/Blog/devextreme-confirmation-delete-person.png" alt="Confirmation message" class="img-thumbnail" />

If we click Yes, it simply calls **DeletePerson** method of **PhoneBookController**.

# Edit Mode For People

First of all, we create the necessary DTOs to transfer people's id, name, surname and e-mail.

```cs
public class EditPersonInput : EntityDto
{
    public string Name { get; set; }

    public string Surname { get; set; }

    public string EmailAddress { get; set; }
}
```

Then create the functions in **PersonAppService** for editing people:  

```csharp
public async Task EditPerson(EditPersonInput input)
{
    var person = await _personRepository.GetAsync(input.Id);
    if (input.Name != null)
    {
        person.Name = input.Name;
    }

    if (input.Surname != null)
    {
        person.Surname = input.Surname;
    }

    if (input.EmailAddress != null)
    {
        person.EmailAddress = input.EmailAddress;
    }

    await _personRepository.UpdateAsync(person);
}
```

Then add **UpdatePerson** method in **PhoneBookController**

```cs
[HttpPut]
public async Task UpdatePerson(int key, string values)
{
    var editPersonInput = new EditPersonInput {Id = key};
    JsonConvert.PopulateObject(values, editPersonInput);

    await _personAppService.EditPerson(editPersonInput);
}
```

## View

Update **Index.cshtml** as seen below.

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
                        .Editing(editing => {
                            editing.Mode(GridEditMode.Row);
                            editing.AllowAdding(true);
                            editing.AllowDeleting(true);
                            editing.AllowUpdating(true);
                        })
                        .DataSource(d => d.Mvc()
                            .Controller("PhoneBook")
                            .LoadAction("LoadPeople")
                            .InsertAction("CreatePerson")
                            .UpdateAction("UpdatePerson")
                            .DeleteAction("DeletePerson")
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

<img src="/Images/Blog/devextreme-confirmation-edit-person.png" alt="Phonebook peoples" class="img-thumbnail"/>

# Multi-Tenancy

We have built a fully functional application until here. Now, we will
see how to convert it to a multi-tenant application easily.

## Enable Multi Tenancy

We disabled multi-tenancy at the beginning of this document. Now,
re-enabling it in PhoneBookDemoConsts class:

```csharp
public const bool MultiTenancyEnabled = true;
```

## Make Entities Multi Tenant

In a multi-tenant application, a tenant's entities should be isolated by
other tenants. For this example project, every tenant should have own
phone book with isolated people and phone numbers.

When we implement IMustHaveTenant interface, ABP automatically [filters
data](https://aspnetboilerplate.com/Pages/Documents/Data-Filters) based
on current Tenant, while retrieving entities from database. So, we
should declare that Person entity must have a tenant using
**IMustHaveTenant** interface:

```csharp
public class Person : FullAuditedEntity, IMustHaveTenant
{
    public virtual int TenantId { get; set; }

    //...other properties
}
```

We may want to add IMustHaveTenant interface to also Phone entity. This
is needed if we directly use phone repository to get phones. In this
sample project, it's not needed.

Since entities have changed, we should create a new database migration:

```
Add-Migration "Implemented_IMustHaveTenant_For_Person"
```

This command creates a new code-first database migration. The migration
class adds an annotation this is needed for automatic filtering. We
don't have to know what it is since it's done automatically. And also it
adds a TenantId column to PbPersons table as shown below:

```csharp
migrationBuilder.AddColumn<int>(name: "TenantId", table: "PbPersons", nullable: false, defaultValue: 1);
```

I added **defaultValue as 1** to AddColumn options. Thus, current people
are automatically assigned to **default tenant** (default tenant's id is
1).

Now, we can update the database again:

```
Update-Database
```

# Running the Application

**It's finished**! We can test the application. Run the project, **login** as the **host admin** (click Change link and clear tenancy name) shown below:

<img src="/Images/Blog/login-as-host-5.png" alt="Login as host" class="img-thumbnail" />

Default **admin** password is **123qwe**. After login, we see the **tenant list** which only contains a **default** tenant. We can create a new tenant:

<img src="/Images/Blog/create-tenant-7.png" alt="Creating tenant" class="img-thumbnail" />

I created a new tenant named **trio**. Now, tenant list has two tenants:

<img src="/Images/Blog/tenant-list-with-2-tenant-2.png" alt="Tenant list" class="img-thumbnail" />

I can **logout** and login as **trio** tenant **admin** (Change current tenant to trio):

<img src="/Images/Blog/login-as-trio3.png" alt="Login as tenant admin" class="img-thumbnail" />

After login, we see that phone book is empty:

<img src="/Images/Blog/phonebook-empty2.png" alt="Empty phonebook of new tenant" class="img-thumbnail" />

It's empty because trio tenant has a completely isolated people list.
You can add people here, logout and login as different tenants (you can
login as default tenant for example). You will see that each tenant has
an isolated phone book and can not see other's people.

# Conclusion

In this document, we built a complete example that covers most parts of the ASP.NET Zero system. We hope that it will help you to build your own
application.

We intentionally used different approaches for similar tasks to show you different styles of development. ASP.NET Zero provides an architecture but does not restrict you. You can make your own style development.

## Code Generation

Using **ASP.NET Zero Power Tools**, you can speed up your development.

See [documentation](https://aspnetzero.com/Documents/Development-Guide-Rad-Tool) to learn how to use it.

### Source Code

You should [purchase](https://aspnetzero.com/Pricing) ASP.NET Zero in order to get **source
code**. After purchasing, you can get the sample project from private
Github repository: <https://github.com/aspnetzero/aspnet-zero-samples>