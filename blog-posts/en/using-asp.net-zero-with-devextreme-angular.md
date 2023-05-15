# Using ASP.NET Zero with DevExtreme MVC

In this document, we will create a sample **phonebook application**
based on ASP.NET Zero (ASP.NET Core & Angular version) step by step.
After all steps, we will have a multi-tenant, localized, authorized,
configurable, testable... application.

## Creating & Running The Project

We're creating and downloading the solution named "**Acme.PhoneBookDemo**" as described in [Getting Started](Getting-Started-Angular) document. Please follow the getting started document, run the application, login as default tenant
admin (select `Default` as tenancy name, use `admin` as username and `123qwe` as the password) and see the dashboard below:

<img src="/Images/Blog/dashboard1.png" alt="Dashboard" style="width:100.0%" />

Logout from the application for now. We will make our application **single-tenant** (we will convert it to multi-tenant later). So, we open **PhoneBookDemoConsts** class in the  **Acme.PhoneBookDemo.Core.Shared** project and disable multi-tenancy as shown below:

```c#
public class PhoneBookDemoConsts
{
    public const string LocalizationSourceName = "PhoneBookDemo";

    public const string ConnectionStringName = "Default";

    public const bool MultiTenancyEnabled = false;

    public const int PaymentCacheDurationInMinutes = 30;
}
```

**Note:** If you log in before changing **MultiTenancyEnabled** to false, you might be get login error. To overcome this, you should remove cookies.

# Adding a New Menu Item

Let's begin from UI and create a new page named "**Phone book**".

## Defining a Menu Item

Open **src\\app\\shared\\layout\\nav\\app-navigation.service.ts** in the client side (**Acme.PhoneBookDemo.AngularUI**) which defines menu items in the application. Create new menu item as shown below (You can add it right after the dashboard menu item).

```csharp
new AppMenuItem("PhoneBook", null, "flaticon-book", "/app/main/phonebook")
```

**PhoneBook** is the menu name (will localize below), **null** is for permission name (will set it later), **flaticon-book** is just an arbitrary icon class (from [this set](http://keenthemes.com/metronic/preview/?page=components/icons/flaticon&demo=default)) and **/phonebook** is the Angular route.

If you run the application, you can see a new menu item on the left menu, but it won't work (it redirects to default route) if you click to the menu item, since we haven't defined the Angular route yet.

## Localize Menu Item Display Name

Localization strings are defined in **XML** files in **.Core** project in server side as shown below:

<img src="/Images/Blog/localization-files-4.png" alt="Localization files" class="img-thumbnail" />

Open PhoneBookDemo.xml (the **default**, **English** localization dictionary) and add the following line:

```xml
<text name="PhoneBook">Phone Book</text>
```

If we don't define "PhoneBook"s value for other localization dictionaries, default value is shown in all languages. For example, we can define it also for Turkish in `PhoneBookDmo-tr.xml` file:

```xml
<text name="PhoneBook">Telefon Rehberi</text>
```

Note: Any change in server side (including change localization texts) requires recycle of the server application. We suggest to use Ctrl+F5 if you don't need to debugging for a faster startup. In that case, it's
enough to make a re-build to recycle the application.

## Angular Route

Angular has a powerful URL routing system. ASP.NET Zero has defined routes in a few places (for modularity, see [main menu & layout](Features-Angular-Main-Menu-Layout.md)). We want to add phone book page to the main module. So, open **src\\app\\main\\main-routing.module.ts** in the client side and add a new route just below to the dashboard:

```json
{
    path: 'phonebook',
    loadChildren: () => import('./phonebook/phonebook.module').then(m => m.PhonebookModule)
}
```

We get an error since we haven't defined PhoneBookModule yet. Also, we ignored permission for now (will implement later).

# Creating the PhoneBook Component

Create a **phonebook** folder inside **src/app/main** folder and add a
new typescript file (**phonebook.component.ts**) in the phonebook folder
as shown below:

```javascript
import { Component, Injector } from '@angular/core';
import { AppComponentBase } from '@shared/common/app-component-base';
import { appModuleAnimation } from '@shared/animations/routerTransition';

@Component({
    templateUrl: './phonebook.component.html',
    animations: [appModuleAnimation()]
})

export class PhoneBookComponent extends AppComponentBase {
    constructor(
        injector: Injector
    ) {
        super(injector);
    }
}
```

We inherited from **AppComponentBase** which provides some common
functions and fields (like localization and access control) for us. It's
not required, but makes our job easier.

As we declared in **phonebook.component.ts** we should create a
**phonebook.component.html** view in the same phonebook folder:

```html
<div [@routerTransition]>
    <div class="kt-content  kt-grid__item kt-grid__item--fluid kt-grid kt-grid--hor">
        <div class="kt-subheader kt-grid__item">
            <div [class]="containerClass">
                <div class="kt-subheader__main">
                    <h3 class="kt-subheader__title">
                        <span>{{"PhoneBook" | localize}}</span>
                    </h3>
                </div>
            </div>
        </div>
        <div [class]="containerClass + ' kt-grid__item kt-grid__item--fluid'"
            <div class="kt-portlet kt-portlet--mobile">
                <div class="kt-portlet__body  kt-portlet__body--fit">
                    <p>PHONE BOOK CONTENT COMES HERE!</p>
                </div>
            </div>
        </div>
    </div>
</div>
```

**l** (lower case 'L') function comes from **AppComponentBase** and used
to easily localize texts. **\@routerTransition** attribute is
required for page transition animation.

Now we should create a **phonebook.module.ts** and **phonebook-routing.module.ts**  view in the same phonebook folder:

*phonebook-routing.module.ts*

```typescript
import {NgModule} from '@angular/core';
import {RouterModule, Routes} from '@angular/router';
import {PhoneBookComponent} from './phonebook.component';

const routes: Routes = [{
    path: '',
    component: PhoneBookComponent,
    pathMatch: 'full'
}];

@NgModule({
    imports: [RouterModule.forChild(routes)],
    exports: [RouterModule],
})
export class PhoneBookRoutingModule {}
```

*phonebook.module.ts*

```typescript
import {NgModule} from '@angular/core';
import {AppSharedModule} from '@app/shared/app-shared.module';
import {PhoneBookRoutingModule} from './phonebook-routing.module';
import {PhoneBookComponent} from './phonebook.component';

@NgModule({
    declarations: [PhoneBookComponent],
    imports: [AppSharedModule, PhoneBookRoutingModule]
})
export class PhoneBookModule {}
```

Now, we can refresh the page to see the new
added page:

<img src="/Images/Blog/phonebook-empty-ng2.png" alt="Phonebook empty" class="img-thumbnail" style="width:100.0%" />

Note: Angular-cli automatically re-compiles and refreshes the page when
any changes made to any file in the application.

# Creating Person Entity

We define entities in **.Core** (domain) project (in server side). We
can define a **Person** entity (mapped to **PbPersons** table in
database) to represent a person in phone book as shown below (I created
in a new folder/namespace named PhoneBook):

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
using Abp.Domain.Entities.Auditing;

namespace Acme.PhoneBookDemo.PhoneBook
{
    [Table("PbPersons")]
    public class Person : FullAuditedEntity
    {
        public const int MaxNameLength = 32;
        public const int MaxSurnameLength = 32;
        public const int MaxEmailAddressLength = 255;

        [Required]
        [MaxLength(MaxNameLength)]
        public virtual string Name { get; set; }

        [Required]
        [MaxLength(MaxSurnameLength)]
        public virtual string Surname { get; set; }

        [MaxLength(MaxEmailAddressLength)]
        public virtual string EmailAddress { get; set; }
    }
}
```

Person's **primary key** type is **int** (as default). It inherits
**FullAuditedEntity** that contains **creation**, **modification** and
**deletion** audit properties. It's also **soft-delete**. When we delete
a person, it's not deleted by database but marked as deleted (see
[entity](https://aspnetboilerplate.com/Pages/Documents/Entities) and
[data
filters](https://aspnetboilerplate.com/Pages/Documents/Data-Filters)
documentations for more information). We created consts for
**MaxLength** properties. This is a good practice since we will use same
values later.

We add a DbSet property for Person entity to **PhoneBookDemoDbContext**
class defined in **.EntityFrameworkCore** project.

```csharp
public class PhoneBookDemoDbContext : AbpZeroDbContext<Tenant, Role, User, PhoneBookDemoDbContext>
{
    public virtual DbSet<Person> Persons { get; set; }

    //...other code
}
```

# Database Migrations for Person

We use **EntityFramework Code-First migrations** to migrate database
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

<img src="/Images/Blog/phonebook-tables-spa.png" alt="Phonebook tables" class="img-thumbnail" />

But this new table is empty. In ASP.NET Zero, there are some classes to fill initial data for users and settings:

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
        //existing code...
        new InitialPeopleCreator(_context).Create();

        _context.SaveChanges();
    }
}
```

We run our project again, it runs seed and adds two people to PbPersons
table:

<img src="/Images/Blog/phonebook-persons-table-initial-data.png" alt="Persons initial data" class="img-thumbnail" width="720" height="50" />

# Creating Unit Tests for Person Application Service

You can skip this section if you are not interested in **automated
testing**.

By writing unit test, we can test **PersonAppService.GetPeople** method
without creating a user interface. We write unit test in .**Tests**
project in the solution.

## MultiTenancy In Tests

Since we disabled multitenancy, we should disable it for unit tests too.
Open **PhoneBookDemoConsts** class in the Acme.PhoneBook.Core project
and set "**MultiTenancyEnabled**" to false. After a rebuild and run unit
tests, you will see that some tests are skipped (those are related to
multitenancy).

Let's create first test to verify getting people without any filter:

```csharp
using Acme.PhoneBookDemo.People;
using Acme.PhoneBookDemo.People.Dtos;
using Shouldly;
using Xunit;

namespace Acme.PhoneBookDemo.Tests.People
{
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
}
```

We derived test class from **AppTestBase**. AppTestBase class
initializes all system, creates an in-memory fake database, seeds
initial data (that we created before) to database and logins to
application as admin. So, this is actually an **integration test** since
it tests all server-side code from entity framework mapping to
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

<img src="/Images/Blog/phone-book-unit-test-success-2.png" alt="xUnit unit test success" class="img-thumbnail" />

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

# Creating Person Application Service

An Application Service is used from client (presentation layer) to perform operations (use cases) of the application.

Application service interface and DTOs are located in **.Application.Shared** project. We are creating an application service to get people from the server. So, we're first creating an **interface** to define the person application service (while this interface is optional, we suggest you to create it):

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

**CustomDtoMapper.cs** is used to create mapping from **Person** to **PersonListDto**. **FullAuditedEntityDto** is inherited to implement audit properties automatically. See [application service](https://aspnetboilerplate.com/Pages/Documents/Application-Services) and [DTO](https://aspnetboilerplate.com/Pages/Documents/Data-Transfer-Objects) documentations for more information. We are adding the following mappings.

```csharp
...
// PhoneBook (we will comment out other lines when the new DTOs are added)
configuration.CreateMap<Person, PersonListDto>();
//configuration.CreateMap<AddPhoneInput, Phone>();
//configuration.CreateMap<CreatePersonInput, Person>();
//configuration.CreateMap<Person, GetPersonForEditOutput>();
//configuration.CreateMap<Phone, PhoneInPersonListDto>();
```

After defining interface, we can implement it as shown below: (in **.Application** project)

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

**WhereIf** is an extension method here (defined in Abp.Linq.Extensions namespace). It performs Where condition, only if filter is not null or empty. **IsNullOrEmpty** is also an extension method (defined in Abp.Extensions namespace). ABP has many similar shortcut extension methods. **ObjectMapper.Map** method automatically converts list of Person entities to list of PersonListDto objects with using configurations in **CustomDtoMapper.cs** in **.Application** project.

### Connection & Transaction Management

We don't manually open database connection or start/commit transactions manually. It's automatically done with ABP framework's Unit Of Work system. See [UOW documentation](https://aspnetboilerplate.com/Pages/Documents/Unit-Of-Work) for more.

### Exception Handling

We don't handle exceptions manually (using a try-catch block). Because ABP framework automatically handles all exceptions on the web layer and returns appropriate error messages to the client. It then handles errors on the client and shows needed error information to the user. See [exception handling document](https://aspnetboilerplate.com/Pages/Documents/Handling-Exceptions) for more.

# Using GetPeople Method From Angular Component

Now, we can switch to the client side and use GetPeople method to show a
list of people on the UI.

## Service Proxy Generation

First, run (prefer Ctrl+F5 to be faster) the server side application
(.Web.Host project). Then run **nswag/refresh.bat** file on the client
side to re-generate service proxy classes (they are used to call server
side service methods).

Since we added a new service, we should add it to
**src/shared/service-proxies/service-proxy.module.ts**. Just open it and
add **ApiServiceProxies.PersonServiceProxy** to the providers array.
This step is only required when we add a new service. If we change an
existing service, it's not needed.

## Angular-Cli Watcher

Sometimes angular-cli can not understand the file changes. In that case,
stop it and re-run **npm start** command.

## PhoneBookComponent Typescript Class

Change **phonebook.component.ts** as like below:

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

We inject **PersonServiceProxy**, call its **getPeople** method and **subscribe** to get the result. We do this in **ngOnInit** function (defined in Angular's **OnInit** interface). Assigned returned items to the **people** class member.

## Rendering People In Angular View

Now, we can use this people member from the view,
**phonebook.component.html**:

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

See the result:

<img src="/Images/Blog/phonebook-people-view-angular-1.png" alt="Phonebook peoples" class="img-thumbnail" />

We successfully retrieved list of people from database to the page.

# Creating a New Person

Next step is to create a modal to add a new item to phone book.

## Add a CreatePerson Method to PersonAppService

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

Then we add configuration for AutoMapper into CustomDtoMapper.cs like below:

```csharp
configuration.CreateMap<CreatePersonInput, Person>();
```

**CreatePersonInput** is mapped to **Person** entity (comment out
related line in CustomDtoMapper.cs and we will use mapping below).
All properties are decorated with **data annotation attributes**
to provide automatic
**[validation](https://aspnetboilerplate.com/Pages/Documents/Validating-Data-Transfer-Objects)**.
Notice that we use same consts defined in **PersonConsts.cs** in
**.Core.Shared** project for **MaxLength** properties. After adding this
class, you can remove consts from **Person** entity and use this new
consts class.

```csharp
public class PersonConsts
{
    public const int MaxNameLength = 32;
    public const int MaxSurnameLength = 32;
    public const int MaxEmailAddressLength = 255;
}
```

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

We did not set **Surname** property of CreatePersonInput despite it being **required**. So, it throws **AbpValidationException** automatically. Also, we can not send null to CreatePerson method since validation system also checks it. This test calls CreatePerson with invalid arguments and asserts that it throws AbpValidationException. See [validation document](https://aspnetboilerplate.com/Pages/Documents/Validating-Data-Transfer-Objects) for more information.

# Adding Create New Person To Index

First of all, we should use **nswag/refresh.bat** to re-generate service-proxies. This will generate the code that is needed to call PersonAppService.**CreatePerson** method from client side. Notice that you should rebuild & run the server side application before re-generating the proxy scripts.

Go to **phonebook.component.ts** and add insert option to `CustomStore`

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
        this.init();
    }

    init() {
        this.refreshMode = "full";

        this.dataSource = new CustomStore({
            key: "id",
            load: (loadOptions) => {
                return this._personService.getPeople("").toPromise();
            },
            insert: (values) => {
                return this._personService.createPerson(values).toPromise()
            }
        });
    }
}
```

Go to **phonebook.component.html** and add `dxo-editing` tag with `allowAdding` true option:

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
                        <dxo-editing
                            mode="row"
                            [refreshMode]="refreshMode"
                            [allowAdding]="true">
                        </dxo-editing>

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

The result:

<img src="/Images/Blog/phonebook-create-person-view-1.png" alt="Create Person Dialog" class="img-thumbnail" />

# Deleting a Person

## Application Service

Let's add a DeletePerson method to the server
side. We are adding it to the service interface, **IPersonAppService:**:

```csharp
Task DeletePerson(EntityDto input);
```

**EntityDto** is a shortcut of ABP if we only get an id value. Implementation (in **PersonAppService**) is very simple:

```csharp
public async Task DeletePerson(EntityDto input)
{
    await _personRepository.DeleteAsync(input.Id);
}
```

## Service Proxy Generation

Since we changed server side services, we should re-generate the client-side service proxies via NSwag. Make server side running and userefresh.bat as we did before.

## Component Script

Go to **phonebook.component.ts** and add delete option to `CustomStore`:

```typescript
import {Component, Injector} from '@angular/core';
import {AppComponentBase} from '@shared/common/app-component-base';
import {appModuleAnimation} from '@shared/animations/routerTransition';
import {EditPersonInput, PersonServiceProxy} from "@shared/service-proxies/service-proxies";
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
        this.init();
    }

    init() {
        this.refreshMode = "full";

        this.dataSource = new CustomStore({
            key: "id",
            load: (loadOptions) => {
                return this._personService.getPeople("").toPromise();
            },
            insert: (values) => {
                return this._personService.createPerson(values).toPromise()
            },
            remove: (key) => {
                return this._personService.deletePerson(key).toPromise();
            }
        });
    }
}

```

Go to **phonebook.component.html** and add `dxo-editing` tag with `allowDeleting` true option:

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
                        <dxo-editing
                            mode="row"
                            [refreshMode]="refreshMode"
                            [allowAdding]="true"
                            [allowDeleting]="true">
                        </dxo-editing>

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

It first shows a confirmation message when we click the delete button:

<img src="/Images/Blog/phonebook-delete-person-view-1.png" alt="Confirmation message" class="img-thumbnail" />

<img src="/Images/Blog/phonebook-delete-person-view-2.png" alt="Confirmation message" class="img-thumbnail" />

# Editing a Person

## Application Service

Let's add a DeletePerson method to the server
side. We are adding it to the service interface, **IPersonAppService:**:

```csharp
Task EditPerson(EditPersonInput input);
```

And add the necessary DTO to transfer person's id, name, surname, and email.

```csharp
public class EditPersonInput: EntityDto
{
    public string Name {get;set;}
    public string Surname {get;set;}
    public string EmailAddress {get;set;}
}
```

**EntityDto** is a shortcut of ABP if we only get an id value. Implementation (in **PersonAppService**) is very simple:

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

## Service Proxy Generation

Since we changed server side services, we should re-generate the client-side service proxies via NSwag. Make server side running and userefresh.bat as we did before.

## Component Script

Go to **phonebook.component.ts** and add update option to `CustomStore`:

```typescript
import {Component, Injector} from '@angular/core';
import {AppComponentBase} from '@shared/common/app-component-base';
import {appModuleAnimation} from '@shared/animations/routerTransition';
import {EditPersonInput, PersonServiceProxy} from "@shared/service-proxies/service-proxies";
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
            },
            insert: (values) => {
                return this._personService.createPerson(values).toPromise()
            },
            update: (key, values) => {
                return this._personService.editPerson(new EditPersonInput({
                    id: key,
                    name: values.name,
                    emailAddress: values.emailAddress,
                    surname: values.surname
                })).toPromise();
            },
            remove: (key) => {
                return this._personService.deletePerson(key).toPromise();
            }
        });
    }
}

```

Go to **phonebook.component.html** and add `dxo-editing` tag with `allowUpdating` true option:

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
                        <dxo-editing
                            mode="row"
                            [refreshMode]="refreshMode"
                            [allowAdding]="true"
                            [allowUpdating]="true"
                            [allowDeleting]="true">
                        </dxo-editing>

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

It first shows a confirmation message when we click the delete button:

<img src="/Images/Blog/phonebook-edit-person-view-1.png" alt="Confirmation message" class="img-thumbnail" />

<img src="/Images/Blog/phonebook-edit-person-view-2.png" alt="Confirmation message" class="img-thumbnail" />

# Multi Tenancy

We have built a fully functional application until here. Now, we will
see how to convert it to a multi-tenant application easily. Logout from
the application before any change.

## Enable Multi Tenancy

We disabled multi-tenancy at the beginning of this document. Now,
re-enabling it in **PhoneBookDemoConsts** class:

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
don't have to know what it is since it's done automatically. It also
adds a **TenantId** column to PbPersons table as shown below:

```csharp
migrationBuilder.AddColumn<int>(name: "TenantId",table: "PbPersons",nullable: false,defaultValue: 1);
```

I added **defaultValue as 1** to AddColumn options. Thus, current people
are automatically assigned to **default tenant** (default tenant's id is
always 1).

Now, we can update the database again:

```
Update-Database
```

# Running the Application

**It's finished**! We can test the application. Run the project, **login** as the **host admin** (click Change link and clear tenancy name) shown below:

<img src="/Images/Blog/login-as-host-3.png" alt="Login as host" class="img-thumbnail" style="width:100.0%" />

Default **admin** password is **123qwe**. After login, we see the **tenant list** which only contains a **default** tenant. We can create a new tenant:

<img src="/Images/Blog/create-tenant-6.png" alt="Creating tenant" class="img-thumbnail" />

I created a new tenant named **trio**. Now, tenant list has two tenants:

<img src="/Images/Blog/tenant-management-phonebook-2.png" alt="Tenant management page" class="img-thumbnail" style="width:100.0%" />

I can **logout** and login as **trio** tenant **admin** (Change current tenant to trio):

<img src="/Images/Blog/login-as-trio1.png" alt="Login as tenant admin" class="img-thumbnail" style="width:100.0%" />

After login, we see that phone book is empty:

<img src="/Images/Blog/phonebook-empty1.png" alt="Empty phonebook of new tenant" class="img-thumbnail" />

It's empty because trio tenant has a completely isolated people list. You can add people here, logout and login as different tenants (you can login as default tenant for example). You will see that each tenant has an isolated phone book and can not see other's people.

# Conclusion

In this document, we built a complete example that covers most parts of
the ASP.NET Zero system. We hope that it will help you to build your own
application.

We intentionally used different approaches for similar tasks to show you
different styles of development. ASP.NET Zero provides an architecture
but does not restrict you. You can make your own style development.


## Code Generation

Using new **ASP.NET Zero Power Tools**, you can speed up your development.

See [documentation](https://aspnetzero.com/Documents/Development-Guide-Rad-Tool) to learn how to use it.

### Source Code

You should [purchase](https://aspnetzero.com/Pricing) ASP.NET Zero in order to get **source
code**. After purchasing, you can get the sample project from private
Github repository: <https://github.com/aspnetzero/aspnet-zero-samples>