# Using ASP.NET Zero with DevExtreme Angular Part 1

In this tutorial, we will guide you through the process of creating a robust and **feature-rich** phonebook application using ASP.NET Zero, which is built on ASP.NET Core and Angular. By following the steps outlined here, you will learn how to develop a **multi-tenant** application that supports **localization**, **authorization**, **configurability**, and **testability**.

ASP.NET Zero provides a solid foundation for building **enterprise-level** applications with its **comprehensive** set of **pre-built features** and functionalities. By leveraging the power of ASP.NET Core on the server-side and Angular on the client-side, you can create a **modern** and **responsive** web application.

By the end of this tutorial, you will have a solid understanding of how to use ASP.NET Zero Angular and DevExtreme together to develop **powerful** applications that are both **user-friendly** and highly functional. So, let's dive in and start building our phonebook application step by step.

## Creating & Running The Project

We're creating and downloading the solution named `Acme.PhoneBookDemo` as described in [Getting Started]([Getting-Started-Angular](https://docs.aspnetzero.com/en/aspnet-core-angular/latest/Getting-Started-Angular)) document. Please follow the getting started document, run the application, login as default tenant admin (select `Default` as tenancy name, use `admin` as username and `123qwe` as the password) and see the dashboard below:

<img src="/Images/Blog/dashboard1.png" alt="Dashboard" style="width:100.0%" />

Logout from the application for now. We will make our application **single-tenant** (we will convert it to multi-tenant later). So, we open `PhoneBookDemoConsts` class in the  `Acme.PhoneBookDemo.Core.Shared` project and disable multi-tenancy as shown below:

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

## Adding a New Menu Item

Let's begin from UI and create a new page named "**Phone book**".

### Defining a Menu Item

Open `*\src\app\shared\layout\nav\app-navigation.service.ts` in the client side (`Acme.PhoneBookDemo.AngularUI`) which defines menu items in the application. Create new menu item as shown below (You can add it right after the dashboard menu item).

```csharp
new AppMenuItem("PhoneBook", null, "flaticon-book", "/app/main/phonebook")
```

`PhoneBook` is the menu name (will localize below), `null` is for permission name (will set it later), `flaticon-book` is just an arbitrary icon class (from [this set](http://keenthemes.com/metronic/preview/?page=components/icons/flaticon&demo=default)) and `/phonebook` is the Angular route.

If you run the application, you can see a new menu item on the left menu, but it won't work (it redirects to default route) if you click to the menu item, since we haven't defined the Angular route yet.

### Localize Menu Item Display Name

Localization strings are defined in **XML** files in `.Core` project in server side as shown below:

<img src="/Images/Blog/localization-files-4.png" alt="Localization files" class="img-thumbnail" />

Open `PhoneBookDemo.xml` (the **default**, **English** localization dictionary) and add the following line:

```xml
<text name="PhoneBook">Phone Book</text>
```

If we don't define "PhoneBook"s value for other localization dictionaries, default value is shown in all languages. For example, we can define it also for Turkish in `PhoneBookDmo-tr.xml` file:

```xml
<text name="PhoneBook">Telefon Rehberi</text>
```

**Note:** Any change in server side (including change localization texts) requires restart of the server application. We suggest to use **Ctrl+F5** if you don't need to debugging for a faster startup. In that case, it's enough to make a **re-build** to recycle the application.

### Angular Route

Angular has a powerful URL routing system. ASP.NET Zero has defined routes in a few places (for modularity, see [main menu & layout]([Features-Angular-Main-Menu-Layout.md](https://docs.aspnetzero.com/en/aspnet-core-angular/latest/Features-Angular-Main-Menu-Layout))). We want to add phone book page to the main module. So, open `*\src\app\main\main-routing.module.ts` in the client side and add a new route just below to the dashboard:

```ts
{
    path: 'phonebook',
    loadChildren: () => import('./phonebook/phonebook.module').then(m => m.PhonebookModule)
}
```

We get an error since we haven't defined `PhoneBookModule` yet. Also, we ignored permission for now (will implement later).

## Creating the PhoneBook Component

Create a `phonebook` folder inside `*\src\app\main` folder and add a
new typescript file `phonebook.component.ts` as shown below:

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

We inherited from `AppComponentBase` which provides some common functions and fields (like localization and access control) for us. It's not required, but makes our job easier.

As we declared in `phonebook.component.ts` we should create a
`phonebook.component.html` view in the same folder.

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

`l` **(lower case 'L')** function comes from `AppComponentBase` and used to easily localize texts. `@routerTransition` attribute is
required for page transition animation.

Now we should create a `phonebook.module.ts` and `phonebook-routing.module.ts`  view in the same folder:

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

Now, we can refresh the page to see the new added page:

<img src="/Images/Blog/phonebook-empty-ng2.png" alt="Phonebook empty" class="img-thumbnail" style="width:100.0%" />

**Note:** Angular-cli automatically re-compiles and refreshes the page when any changes made to any file in the application.

## Creating Person Entity

We define entities in `.Core` (domain) project (in server side). We
can define a `Person` entity (mapped to `PbPersons` table in
database) to represent a person in phone book as shown below (I created in a new folder/namespace named `PhoneBook`):

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

Person's **primary key** type is **int** (as default). It inherits `FullAuditedEntity` that contains `creation`, `modification` and `deletion` audit properties. It's also `soft-delete`. When we delete a person, it's not deleted by database but marked as deleted (see [entity](https://aspnetboilerplate.com/Pages/Documents/Entities) and [data filters](https://aspnetboilerplate.com/Pages/Documents/Data-Filters) documentations for more information). We created consts for `MaxLength` properties. This is a good practice since we will use same values later.

We add a `DbSet` property for `Person` entity to `PhoneBookDemoDbContext` class defined in `.EntityFrameworkCore` project.

```csharp
public class PhoneBookDemoDbContext : AbpZeroDbContext<Tenant, Role, User, PhoneBookDemoDbContext>
{
    public virtual DbSet<Person> Persons { get; set; }

    //...other code
}
```

## Database Migrations for Person

We use **EntityFramework Code-First migrations** to migrate database schema. Since we added `Person` entity, our DbContext model is changed. So, we should create a **new migration** to create the new table in the database.

### Creating Migration

Open **Package Manager Console**, run the `Add-Migration "Added_Persons_Table"` command as shown below:

<img src="/Images/Blog/phonebook-migrations-core-3.png" alt="Entity Framework Code First Migration" class="img-thumbnail" />

This command will add a **migration class** named `Added_Persons_Table` as shown below:

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

We don't have to know so much about format and rules of this file. But, it's suggested to have a basic understanding of migrations. In the same **Package Manager Console**, we write `Update-Database` command in order to apply the new migration to database. After updating, we can see that `PbPersons` table is added to database.

<img src="/Images/Blog/phonebook-tables-spa.png" alt="Phonebook tables" class="img-thumbnail" />

But this new table is empty. In ASP.NET Zero, there are some classes to fill initial data for users and settings:

<img src="/Images/Blog/aspnet-core-ef-seed-1.png" alt="Seed folders" class="img-thumbnail" />

### Seed Person Data

So, we can add a separated class to fill some people to database as shown below:

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

These type of default data is good since we can also use these data in **unit tests**. Surely, we should be careful about seed data since this code will always be executed in each `PostInitialize` of your `PhoneBookEntityFrameworkCoreModule`. This class (`InitialPeopleCreator`) is created and called in `InitialHostDbBuilder` class. This is not so important, just for a good code organization (see source codes).

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

We run our project again, it runs seed and adds two people to `PbPersons` table:

<img src="/Images/Blog/phonebook-persons-table-initial-data.png" alt="Persons initial data" class="img-thumbnail" width="720" height="50" />

## Conclusion

In this tutorial, we have explored the process of creating a phonebook application using ASP.NET Zero, which is built on ASP.NET Core and Angular, and integrated with DevExtreme components.

### Summary

* We started by creating and setting up the ASP.NET Zero solution, following the instructions in the Getting Started document. We logged in as the default tenant admin and familiarized ourselves with the dashboard.

* We made the application **single-tenant** by disabling multi-tenancy in the `PhoneBookDemoConsts` class.

* Next, we added a new menu item for the phonebook page by defining a menu item in the `AppNavigationService` class. We also localized the menu item display name by adding the necessary entries in the localization XML files.

* We defined the Angular route for the phonebook page by adding a new route in the `main-routing.module.ts` file. We created the corresponding components and modules for the phonebook page and implemented a basic layout and content.

* Moving to the server-side, we created a `Person` entity to represent a person in the phonebook and added it to the `PhoneBookDemoDbContext`.

* We used Entity Framework **Code-First** Migrations to create a new migration for the `Person` entity and applied it to the database. This created the `PbPersons` table in the database.

* Finally, we added seed data to the `Person` table using the `InitialPeopleCreator` class, which was called in the `InitialHostDbBuilder`. This allowed us to have some initial data in the database.

### What's Next?

In the next tutorial, we will add a **application service** to manage the phonebook. We will also add a **unit test** for the application service. Finally, we will add a **new page** to the **Angular** application to display the phonebook.

See the [next part](https://aspnetzero.com/blog/using-asp.net-zero-with-devextreme-angular-part-2) of this tutorial.