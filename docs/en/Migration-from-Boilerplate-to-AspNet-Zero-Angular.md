# Migration Guide: Transitioning from ASP.NET Boilerplate to ASP.NET Zero

This is a step-by-step guide to migrate your Angular application from ASP.NET Boilerplate to ASP.NET Zero.

## Introduction

The purpose of this migration guide is to provide a comprehensive, step-by-step process for migrating from ASP.NET Boilerplate to ASP.NET Zero in the context of an Angular application. This guide aims to help developers understand the differences between the two applications, ensure a smooth migration, and leverage the advanced features of ASP.NET Zero to improve application performance and maintainability.

## Creating and Migrating Entities

Let's consider an entity called `Book` that was previously created in an ASP.NET Boilerplate application as follows:

```c#
public class Book : Entity<long>, IHasCreationTime
{
    public string Name { get; set; }  
    public string Description { get; set; }
    public DateTime CreationTime { get; set; }

    public Book()
    {
        CreationTime = Clock.Now;
    }
}
```

### Transitioning to ASP.NET Zero

When migrating this entity to ASP.NET Zero, you can create a new class within the `*.Core` project, just like you did in your ASP.NET Boilerplate application. This ensures consistency and simplifies integration into your ASP.NET Zero solution.

## Configuring DbContext

To integrate the `*DbContext` added entity in ASP.NET Boilerplate into ASP.NET Zero, add it to the `*DbContext` class in the `*.EntityFrameworkCore` project as shown below:

```c#
public class AngularProjectDemoDbContext : AbpZeroDbContext<Tenant, Role, User, AngularProjectDemoDbContext>, IOpenIddictDbContext
{
    /* Define an DbSet for each entity of the application */

    public virtual DbSet<Book> Books { get; }

    ...

    public AngularProjectDemoDbContext(DbContextOptions<AngularProjectDemoDbContext> options)
        : base(options)
    {
    }

    ...
}
```

## Database Migrations

To add the entity added to the `*DbContext` to the database table, first, we need to create a migration. You can follow the exact same steps applied in ASP.NET Boilerplate using the **Package Manager Console**. After opening it, select the project to be applied in the Default Project section and create a migration with the specified command.

![Adding Migration in Package Manager Console](images/adding-migration.png)

After applying this command, it is sufficient to apply the following command to reflect the created migration to the database.

![Update Database in Package Manager Console](images/updating-database.png)

### Migrating Databases Between Applications

When transitioning from an ASP.NET Boilerplate application to an ASP.NET Zero application, it is essential to consider how the database migration will be handled. Since both frameworks typically use Microsoft SQL Server (MSSQL) as the database management system, the migration process can be streamlined by leveraging existing migration tools and practices. Given that both frameworks utilize the ABP framework, many of the tables will have similar characteristics, which simplifies the migration process.

Here are the key steps to ensure a smooth database migration:

- Analyze Existing Database Schema: Start by analyzing the existing database schema in the ASP.NET Boilerplate application. Identify the entities, relationships, and constraints that need to be migrated.

- Generate Migrations in ASP.NET Zero: Use the Entity Framework Core migration tools to generate the necessary migrations in the ASP.NET Zero application. This involves creating an initial migration that mirrors the existing database schema.

- Data Migration: Ensure that data is accurately migrated from the ASP.NET Boilerplate database to the ASP.NET Zero database. 

- Test the Migration: Thoroughly test the migration process in a staging environment. Verify that all entities and data have been correctly migrated and that the application functions as expected.

For more detailed information and best practices on using Entity Framework Core with MSSQL, refer to the official Microsoft documentation on [SQL Server migration](https://learn.microsoft.com/en-us/sql/sql-server/migrate/?view=sql-server-ver16) and [copying databases](https://learn.microsoft.com/en-us/sql/relational-databases/databases/copy-databases-to-other-servers?view=sql-server-ver16). Additionally, [this guide](https://www.sqlshack.com/six-different-methods-to-copy-tables-between-databases-in-sql-server/) provides a comprehensive overview of different methods to copy tables between databases in SQL Server, offering practical examples and steps.

> When migrating data from an ASP.NET Boilerplate database to an ASP.NET Zero database, ensure that all existing data is accurately transferred. Testing the migration process in a staging environment is crucial to ensure that the migration is successful and that the application functions as expected after the migration.

## Implementing Application Services

This chapter covers the implementation of application services in ASP.NET Zero, focusing on creating service interfaces, DTOs (Data Transfer Objects), and their associated services class.

### Creating DTOs and Defining AppService Interfaces

In this section, creating application service classes requires defining necessary interfaces and DTOs. Similar to ASP.NET Boilerplate, however, interfaces and DTOs created here will be managed in the `*.Application.Shared` project instead of `*.Application` for easier organization and accessibility.

**IBookAppService** manages general CRUD operations for the **Book** entity, defined with the following method signatures:

```c#
public interface IBookAppService: IApplicationService
{
    Task<ListResultDto<BookListDto>> GetBooks(GetBooksInput input);

    Task<GetBookForEditOutput> GetBookForEdit(NullableIdDto input);

    Task CreateOrUpdateBook(CreateOrUpdateBookInput input);

    Task DeleteBook(EntityDto input);
}
```

In ASP.NET Zero, DTOs like `CreateOrUpdateBookInput`, `GetBooksInput`, `BookListDto`, and `GetBookForEditOutput` are typically organized under the `*.Application.Shared` project, specifically under a folder structure such as **Books/Dto**. 

You can also use the same dto structure that you created in ASP.NET Boilerplate in the ASP.NET Zero application.

Like ASP.NET Boilerplate, you can create interfaces in ASP.NET Zero in a similar way.

## Managing Permissions

In ASP.NET Zero applications, it is important to manage permissions, as in ASP.NET Boilerplate, to control access to various features and functions within the application. Permissions are typically defined as constants in a central static class called AppPermissions.

```c#
public static class AppPermissions
{
    ...

    public const string Pages_Administration_Books = "Pages.Administration.Books";
    public const string Pages_Administration_Books_Create = "Pages.Administration.Books.Create";
    public const string Pages_Administration_Books_Edit = "Pages.Administration.Books.Edit";
    public const string Pages_Administration_Books_Delete = "Pages.Administration.Books.Delete";

    ...
}
```

In ASP.NET Zero applications, one distinction from ASP.NET Boilerplate in app service classes lies in CRUD operations where permission control is managed. Here, to access general app service functionality, a permission named `Pages_Administration_Books` is created, encompassing permissions for **Create**, **Edit**, and **Delete** operations. For these methods, separate permission checks can optionally be applied in ASP.NET Boilerplate applications as well. This approach aims to facilitate easier management and controlled access, emphasizing structured handling of permissions.

### Implementing App Service Classes

Create an app service class created in an `*.Application` project, as created in ASP.NET Boilerplate, that will implement the **IBookAppService** interface.

```c#
 [AbpAuthorize(AppPermissions.Pages_Administration_Books)]
 public class BookAppService : AngularProjectDemoAppServiceBase, IBookAppService
 {
     private readonly IRepository<Book, long> _bookRepository;

     public BookAppService(IRepository<Book, long> bookRepository)
     {
         _bookRepository = bookRepository;
     }

     public async Task CreateOrUpdateBook(CreateOrUpdateBookInput input)
     {
         if (input.Book.Id.HasValue)
         {
             await UpdateBookAsync(input);
         }
         else
         {
             await CreateBookAsync(input);
         }
     }


     [AbpAuthorize(AppPermissions.Pages_Administration_Books_Delete)]
     public async Task DeleteBook(EntityDto input)
     {
         var book = await _bookRepository.GetAsync(input.Id);

         await _bookRepository.DeleteAsync(book);
     }


     [AbpAuthorize(AppPermissions.Pages_Administration_Books_Create, AppPermissions.Pages_Administration_Books_Edit)]
     public async Task<GetBookForEditOutput> GetBookForEdit(NullableIdDto input)
     {
         BookEditDto bookListDto;

         if (input.Id.HasValue)
         {
             var book = await _bookRepository.GetAsync(input.Id.Value);
             bookListDto = ObjectMapper.Map<BookEditDto>(book);
         }
         else
         {
             bookListDto = new BookEditDto();
         }

         return new GetBookForEditOutput
         {
             Book = bookListDto
         };
     }

     [HttpPost]
     public async Task<ListResultDto<BookListDto>> GetBooks(GetBooksInput input)
     {
         var query = _bookRepository.GetAll();

         query = query
             .WhereIf(!input.Filter.IsNullOrEmpty(),
                 b => b.Name.Contains(input.Filter)
             );

         var books = await query.ToListAsync();

         return new ListResultDto<BookListDto>(ObjectMapper.Map<List<BookListDto>>(books));
     }

     [AbpAuthorize(AppPermissions.Pages_Administration_Books_Edit)]
     protected virtual async Task UpdateBookAsync(CreateOrUpdateBookInput input)
     {
         var book = await _bookRepository.GetAsync(input.Book.Id.Value);
         book.Name = input.Book.Name;
         book.Description = input.Book.Description;
     }

     [AbpAuthorize(AppPermissions.Pages_Administration_Books_Create)]
     protected virtual async Task CreateBookAsync(CreateOrUpdateBookInput input)
     {
         var book = ObjectMapper.Map<Book>(input.Book);

         await _bookRepository.InsertAsync(book);
     } 
 }
```

This **BookAppService** class demonstrates how CRUD operations for books are implemented in an ASP.NET Zero application, following a straightforward approach similar to ASP.NET Boilerplate.

## Navigation and Page Management

In ASP.NET Zero, routing is managed through route modules similar to ASP.NET Boilerplate, but with a key difference: ASP.NET Zero now uses **standalone components** instead of NgModules. Routes are defined in `src\app\admin\admin-routing.module.ts` for admin pages or `src\app\main\main-routing.module.ts` for main application pages.

**app-routing.module (ASP.NET Boilerplate)**

```typescript
@NgModule({
    imports: [
        RouterModule.forChild([
            {
                path: '',
                component: AppComponent,
                children: [
                    //...

                    { path: 'books', component: BooksComponent, data: { permission: 'Pages.Books' }, canActivate: [AppRouteGuard] },

                    //...
                ]
            }
        ])
    ],
    exports: [RouterModule]
})
```

**admin-routing.module or main-routing.module (ASP.NET Zero - Standalone Components)**

```typescript
@NgModule({
    imports: [
        RouterModule.forChild([
            {
                path: '',
                children: [

                    //...
                    
                    {
                        path: 'books',
                        loadComponent: () => import('./books/books.component').then((m) => m.BooksComponent),
                        data: { permission: 'Pages.Administration.Books' },
                    },

                    //...

                ],
            },
        ]),
    ],
    exports: [RouterModule],
})
```

**Key Difference:** ASP.NET Zero uses `loadComponent` instead of `loadChildren`, directly lazy-loading the standalone component instead of loading an entire module.

After defining page routes, to facilitate navigation to these pages via the menu in ASP.NET Zero, similar to ASP.NET Boilerplate's `src\app\layout\sidebar-menu.component.ts` class, ASP.NET Zero employs a similar structure within the `src\app\shared\layout\nav\app-navigation-service.ts` class. You can add navigation items to specific locations within this class based on your usage scenario.

**sidebar-menu.component (ASP.NET Boilerplate)**

```typescript
getMenuItems(): MenuItem[] {
    return [
    //...

    new MenuItem(this.l('Books'), '/app/Books', 'fas fa-books', 'Pages.Books'),

    //...

    ]
}
```


**app-navigation-service (ASP.NET Zero)**

```typescript
getMenu(): AppMenu {
    return new AppMenu('MainMenu', 'MainMenu', [
    //...

    new AppMenuItem('Books', 'Pages.Administration.Books', 'flaticon-books', '/app/main/books'),

    //...
    ])
}
```


## Building the User Interface

The pages and modals created here show similar features. Since it is the Metronic theme used in ASP.NET Zero, it is recommended that you review the link [Metronic Theme](https://preview.keenthemes.com/metronic8/demo1/index.html?mode=light) to adapt it.

### Creating Standalone Components and Views

**Important:** ASP.NET Zero has migrated to **standalone components**, eliminating the need for separate NgModule files for each feature. This is a major architectural difference from ASP.NET Boilerplate.

In ASP.NET Boilerplate, components are declared in `app.module.ts`. In ASP.NET Zero, components are **standalone** and declare their dependencies directly in the `@Component` decorator using the `imports` array.

**app.module.ts (ASP.NET Boilerplate)**

```typescript
@NgModule({
    declarations: [

        //...

        //books
        BooksComponent,
        CreateBookDialogComponent,
        EditBookDialogComponent,

        //...
    ],
})
export class AppModule {}
```

**books.component.ts (ASP.NET Zero - Standalone Component)**

```typescript
@Component({
    selector: 'books',
    templateUrl: './books.component.html',
    animations: [appModuleAnimation()],
    standalone: true,
    imports: [
        CommonModule,
        FormsModule,
        TableModule,
        PaginatorModule,
        SubHeaderComponent,
        LocalizePipe,
    ],
})
export class BooksComponent extends AppComponentBase {
    private readonly _bookService = inject(BookServiceProxy);

    // Component logic here
}
```

**create-or-edit-book-modal.component.ts (ASP.NET Zero - Standalone Modal)**

```typescript
@Component({
    selector: 'createOrEditBookModal',
    templateUrl: './create-or-edit-book-modal.component.html',
    standalone: true,
    imports: [
        CommonModule,
        FormsModule,
        ModalModule,
        LocalizePipe,
        ButtonBusyDirective,
    ],
})
export class CreateOrEditBookModalComponent extends AppComponentBase {
    private readonly _bookService = inject(BookServiceProxy);

    modalSave = output<any>();
    modal = viewChild.required<ModalDirective>('modal');

    // Modal logic here
}
```

**Key Points:**
- **No module files needed:** You don't need to create `book.module.ts` or declare components in `app.module.ts`
- **standalone: true:** All components must have this flag in the `@Component` decorator
- **imports array:** Declare all dependencies (modules, components, pipes, directives) directly in the component
- **Dependency injection:** Use `inject()` function for cleaner dependency injection
- **Modern Angular patterns:** Use `signal()`, `viewChild()`, and `output()` for reactive state management

In ASP.NET Zero, create and edit modals are typically managed in a single modal component (`CreateOrEditBookModalComponent`), unlike ASP.NET Boilerplate which often has separate components for create and edit operations.

The approach and features exhibited in ASP.NET Boilerplate `books.component.ts` differ from ASP.NET Zero.

**Differences**

- **ASP.NET Boilerplate:** 
    - Uses NgModules with `declarations` array for components
    - Manages modals using BsModalService (with BsModalRef)
    - Angular uses HTML table
    - Service connections are made more manually
    - These types of features are often added as a base
    - **abp.message.confirm** and **abp.notify.success** are used

- **ASP.NET Zero:** 
    - Uses **standalone components** with `standalone: true` flag
    - Components declare their own dependencies in `imports` array
    - Uses modern Angular patterns: `inject()`, `signal()`, `viewChild()`, `output()`
    - Manages modals using Angular's **viewChild** mechanism
    - Uses Table and **Paginator** components from the primeng library
    - More automated and structured, additional libraries such as **primeng** are used
    - Excel and file upload features are more advanced and integrated (FileUpload, ExcelColumnSelectionModalComponent)
    - Uses more advanced notification and approval services (NotifyService)
    - Uses advanced filtering and loading mechanisms with primeng
    - Lazy loads components directly with `loadComponent` instead of `loadChildren`


When transitioning from ASP.NET Boilerplate (ABP) to ASP.NET Zero (ABP.Zero), especially in Angular components like `books.component.html`, there are several key differences and considerations:

- **ASP.NET Boilerplate:**
    - Uses NgModules with component declarations
    - Uses a structured layout with specific Angular directives and Bootstrap for UI components
    - Does not include role checks for creating buttons
    - The original Index view is placed directly under the **Views/Books** folder
    - Uses traditional `@ViewChild` decorator syntax

- **ASP.NET Zero:**
    - Uses **standalone components** architecture - no NgModule files needed
    - Components are self-contained with their own `imports` array
    - Often utilizes more integrated and streamlined components provided by the Metronic Theme
    - Uses permission attributes (`isGranted('Pages.Administration.Books.Create')`) for the user creation button
    - The view is moved under **Areas/App/Views/Books** to support modular and multi-tenancy architecture
    - Uses modern Angular signals with `viewChild()`, `signal()`, and `output()` functions

In ASP.NET Boilerplate, the creation and editing modals are managed separately, whereas in ASP.NET Zero, these are handled within a single view under a unified modal management approach.

**Migration Summary for Angular Components:**

1. Convert NgModule based components to standalone components
2. Add `standalone: true` to `@Component` decorator
3. Move all imports from NgModule to component's `imports` array
4. Update routing to use `loadComponent` instead of `loadChildren`
5. Consider adopting modern Angular patterns like `inject()`, `signal()`, and `viewChild()`

## Adding Localization

As in ASP.NET Boilerplate, localization resources in ASP.NET Zero are created under the *.Core project within the Localization folder. In ASP.NET Zero, the source files are located in a folder named after your project, such as AngularProjectDemo. You can add entries to the AngularProjectDemo.xml file or any other language file as shown below:

```xml
<localizationDictionary culture="en">
  <texts>
    ...
    <text name="Books">Books</text>
    <text name="BooksHeaderInfo">Books Header Info</text>
    <text name="CreateNewBook">Create New Book</text>
  </texts>
</localizationDictionary>
```


## Implementing Unit Tests

Unit tests implemented in ASP.NET Boilerplate show minor changes in ASP.NET Zero.

**Main Differences:**

**ASP.NET Boilerplate:** More manual setup with Effort and custom fixtures.
**ASP.NET Zero:** Uses ASP.NET Core’s DI and in-memory database providers for simpler and more consistent setup.

**ASP.NET Boilerplate:** Uses Effort for simulating an in-memory database.
**ASP.NET Zero:** Uses SQL Server or Entity Framework Core’s in-memory database for easier configuration.

**Transition Steps:**

- Transition from base class TestBase to AppTestBase provided by ASP.NET Zero.
- Replace Effort with Entity Framework Core’s in-memory database provider.
- Ensure dependencies are resolved using ASP.NET Core’s DI framework.
- Adapt test methods to utilize ASP.NET Zero’s testing utilities and patterns.


## Conclusion

The migration from ASP.NET Boilerplate to ASP.NET Zero involves several important transitions in various aspects of application development. This process includes changes in the management of models, configuration of app services, unit tests, and most notably, the Angular architecture.

**Key Migration Points:**

- **Angular Architecture:** ASP.NET Zero uses standalone components instead of NgModules, eliminating the need for module files
- **Modern Angular Patterns:** Adoption of `inject()`, `signal()`, `viewChild()`, and `output()` for better type safety and reactivity
- **Routing:** Transition from `loadChildren` (modules) to `loadComponent` (standalone components) for lazy loading
- **UI Components:** Enhanced UI with PrimeNG and Metronic theme integration
- **Development Experience:** Improved with more streamlined and self-contained components

ASP.NET Zero offers advanced features that increase efficiency and scalability by providing a more integrated and streamlined approach to these elements. The adoption of standalone components represents a significant architectural improvement, aligning with modern Angular best practices and reducing boilerplate code.