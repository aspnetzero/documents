# Building a Modular Monolith Application with ASP.NET Zero
In today's software development landscape, the debate between monolithic and microservices architectures is common. However, a middle ground combines the simplicity of monolithic development with the modularity of microservices: the modular monolith. ASP.NET Zero, a powerful framework for building enterprise applications, supports this architecture elegantly. In this blog post, we'll explore how to create a modular monolith application using ASP.NET Zero, complete with an example project and step-by-step instructions.

## What is a Modular Monolith?
A modular monolith is a single application that is logically divided into different modules. Each module is self-contained, with its own domain, but they all run within a single process. This approach combines the best of both worlds: the simplicity and performance of monoliths and the modularity and scalability of microservices.

## Example Project: Building a Library Application
To demonstrate how to build a modular monolith application with ASP.NET Zero, let's create a library management system. Our application will have `BookTracking` module. If you want, you can create more modules like the example here.

In this article, I will focus on how to implement a modular structure. I will not cover the details of the ASP.NET Zero. If you are new to ASP.NET Zero, I recommend checking out the [official documentation](https://docs.aspnetzero.com/).

* First, download a new template from the ASP.NET Zero website. Choose the `ASP.NET Core` option and select the `ASP.NET Core MVC` template.

* In your solution, create a new folder named `modules`.

* Create a subfolder named `BookTracking` inside the `modules` folder.

Let's dive into the steps to create the `BookTracking` module.

### 1. Add Core Project:
Add class library project named `BookTracking.Core` inside `BookTracking` folder. This project will contain the domain entities for the `BookTracking` module.

Add following nuget packages to the `BookTracking.Core` project:

```xml	
<ItemGroup>
    <PackageReference Include="Abp.AspNetZeroCore" Version="5.0.0" />
    <PackageReference Include="Abp.AutoMapper" Version="9.2.2" />
    <PackageReference Include="Abp.ZeroCore.EntityFrameworkCore" Version="9.2.2" />
</ItemGroup>
```

#### Core Module

Add following classes to the `BookTracking.Core` project:

*BookTrackingConsts.cs*
```csharp
namespace BookTracking.Core;

public class BookTrackingConsts
{
    public const string LocalizationSourceName = "BookTracking";
    
    public const bool MultiTenancyEnabled = true;
}
```

*BookTrackingCoreModule* 
```csharp
using System.Reflection;
using Abp.AspNetZeroCore;
using Abp.AutoMapper;
using Abp.Modules;
using Abp.Zero;
using BookTracking.Core.Localization;
using BookTracking.Core.Settings;

namespace BookTracking.Core;

[DependsOn(
    typeof(AbpZeroCoreModule),
    typeof(AbpAutoMapperModule),
    typeof(AbpAspNetZeroCoreModule)
)]
public class BookTrackingCoreModule : AbpModule
{
    public override void PreInitialize()
    {
        Configuration.Settings.Providers.Add<BookTrackingSettingProvider>();

        BookTrackingLocalizationConfigurer.Configure(Configuration.Localization);

        //Enable this line to create a multi-tenant application.
        Configuration.MultiTenancy.IsEnabled = BookTrackingConsts.MultiTenancyEnabled;
    }

    public override void Initialize()
    {
        IocManager.RegisterAssemblyByConvention(Assembly.GetExecutingAssembly());
    }
}
```

#### Settings

Create a new folder named `Settings` in the `BookTracking.Core` project. Add a new class named , `BookTrackingSettingProvider.cs` in the `Settings` folder. 

*BookTrackingSettingProvider.cs*
```csharp
using Abp.Configuration;

namespace BookTracking.Core.Settings;

public class BookTrackingSettingProvider : SettingProvider
{
    public override IEnumerable<SettingDefinition> GetSettingDefinitions(SettingDefinitionProviderContext context)
    {
        return new[]
        {
            new SettingDefinition(
                "BookTrackingBackgroundColor",
                "#1e1e2d"
            ),
            new SettingDefinition(
                "BookTrackingForegroundColor",
                "#fff"
            )
        };
    }
}
```

#### Localization

Create new folder named `Localization` in the `BookTracking.Core` project. Add a new resource file named `BookTracking` in the `Localization` folder. Add the following key-value pairs to the resource file:

*BookTracking.xml*
```xml
<?xml version="1.0" encoding="utf-8" ?>
<localizationDictionary culture="en">
    <texts>
        <text name="Books">Books</text>
        <text name="BookNotFound">Book is not found!</text>
        <text name="BooksHeaderInfo">Books management page</text>
        <text name="Name">Name</text>
        <text name="Author">Author</text>
        <text name="Description">Description</text>
        <text name="ISBN">ISBN</text>
        <text name="TotalPages">Total Pages</text>
        <text name="CreateNewBook">Create new book</text>
        <text name="CreateNewBookHeader">Create New Book</text>
        <text name="EditBookHeader">Edit Book: {0}</text>
        <text name="Actions">Actions</text>
        <text name="Edit">Edit</text>
        <text name="Save">Save</text>
        <text name="Delete">Delete</text>
        <text name="Cancel">Cancel</text>
    </texts>
</localizationDictionary>
```

*BookTracking-tr.xml*
```xml
<?xml version="1.0" encoding="utf-8" ?>
<localizationDictionary culture="tr">
    <texts>
        <text name="Books">Kitaplar</text>
        <text name="BookNotFound">Kitap bulunamadı!</text>
        <text name="BooksHeaderInfo">Kitap yönetimi sayfası</text>
        <text name="Name">İsim</text>
        <text name="Author">Yazar</text>
        <text name="Description">Açıklama</text>
        <text name="ISBN">ISBN</text>
        <text name="TotalPages">Toplam Sayfa</text>
        <text name="CreateNewBook">Yeni kitap oluştur</text>
        <text name="CreateNewBookHeader">Yeni Kitap Oluştur</text>
        <text name="EditBookHeader">Kitabı Düzenle: {0}</text>
        <text name="Actions">Eylemler</text>
        <text name="Edit">Düzenle</text>
        <text name="Save">Kaydet</text>
        <text name="Delete">Sil</text>
        <text name="Cancel">İptal</text>
    </texts>
</localizationDictionary>
```

*BookTrackingLocalizationConfigurer.cs*
```csharp	
using Abp.Configuration.Startup;
using Abp.Localization.Dictionaries;
using Abp.Localization.Dictionaries.Xml;
using Abp.Reflection.Extensions;

namespace BookTracking.Core.Localization
{
    public static class BookTrackingLocalizationConfigurer
    {
        public static void Configure(ILocalizationConfiguration localizationConfiguration)
        {
            localizationConfiguration.Sources.Add(
                new DictionaryBasedLocalizationSource(
                    BookTrackingConsts.LocalizationSourceName,
                    new XmlEmbeddedFileLocalizationDictionaryProvider(
                        typeof(BookTrackingLocalizationConfigurer).GetAssembly(), 
                        "BookTracking.Core.Localization"
                    )
                )
            );
        }
    }
}
```

* Let's configure csproj file of the `BookTracking.Core` project to include the localization files. Add the following code to the `BookTracking.Core.csproj` file:

```xml
<ItemGroup>
    <EmbeddedResource Include="Localization\**\*.xml"/>
</ItemGroup>
```

#### Authorization

Create a new folder named `Authorization` in the `BookTracking.Core` project. Add following classes to the `Authorization` folder:

*BookTrackingAuthorizationProvider.cs*
```csharp
using Abp.Authorization;
using Abp.Configuration.Startup;
using Abp.Localization;
using BookTracking.Core.Authorization;

namespace BookTracking.Application.Authorization;

public class BookTrackingAuthorizationProvider(bool isMultiTenancyEnabled) : AuthorizationProvider
{
    private readonly bool _isMultiTenancyEnabled = isMultiTenancyEnabled;

    public BookTrackingAuthorizationProvider(IMultiTenancyConfig multiTenancyConfig) : this(multiTenancyConfig.IsEnabled)
    {
    }
    
    public override void SetPermissions(IPermissionDefinitionContext context)
    {
        //COMMON PERMISSIONS (FOR BOTH OF TENANTS AND HOST)

        var bookTracking = context.GetPermissionOrNull(BookTrackingPermissions.BookTracking) ?? context.CreatePermission(BookTrackingPermissions.BookTracking, L("BookTracking"));
        
        var book = bookTracking.CreateChildPermission(BookTrackingPermissions.BookTracking_Book, L("Books"));
        book.CreateChildPermission(BookTrackingPermissions.BookTracking_Book_Create, L("CreatingNewBook"));
    }

    private static ILocalizableString L(string name)
    {
        return new LocalizableString(name, "BookTracking");
    }
}
```

*BookTrackingPermissions.cs*
```csharp
namespace BookTracking.Core.Authorization;

public static class BookTrackingPermissions
{
    public const string BookTracking = "BookTracking";
    public const string BookTracking_Book = BookTracking + ".Book";
    public const string BookTracking_Book_Create = BookTracking_Book + ".Create";
}
```

#### Book Entity

Create a new folder named `Books` in the `BookTracking.Core` project. Add a new class named `Book.cs` in the `Books` folder.

```csharp
using Abp.Domain.Entities;

namespace BookTracking.Core.Books;

public class Book : Entity, IMayHaveTenant
{
    public int? TenantId { get; set; }

    public string Name { get; set; }
    
    public string Author { get; set; }
    
    public string Description { get; set; }
    
    public string ISBN { get; set; }
    
    public int TotalPages { get; set; }
}
```

### 2. Add Business Logic Project:
Inside the `BookTracking` folder, add a new class library project named `BookTracking.Application`. This project will handle the business logic for the `BookTracking` module.

Add reference to the `BookTracking.Core` project in the `BookTracking.Application` project.

#### DTO's

* Create a new folder named `Books` in the `BookTracking.Application` project. 

* Create a subfolder named `Dto` inside the `Books` folder.

Add following classes to the `Dto` folder:

*BookListDto.cs*
```csharp
using Abp.Application.Services.Dto;

namespace BookTracking.Application.Books.Dto;

public class BookListDto : EntityDto
{
    public string Name { get; set; }
    
    public string Author { get; set; }
    
    public string Description { get; set; }
    
    public string ISBN { get; set; }
    
    public int TotalPages { get; set; }
}
```

*GetBooksInput.cs*
```csharp
using Abp.Application.Services.Dto;

namespace BookTracking.Application.Books.Dto;

public class GetBooksInput : PagedAndSortedResultRequestDto
{
    // Add your filtering properties here
}
```

#### AppServices

Add following classes to the `Books` folder:

*IBookAppService.cs*
```csharp
using Abp.Application.Services;
using Abp.Application.Services.Dto;
using BookTracking.Application.Books.Dto;

namespace BookTracking.Application.Books;

public interface IBookAppService : IApplicationService
{
    Task<PagedResultDto<BookListDto>> GetBooks(GetBooksInput input);
    
    Task<CreateOrEditBookDto> GetUserForEdit(int id);
    
    Task CreateOrEditBook(CreateOrEditBookDto input);
    
    Task DeleteBook(int id);
}
```

*BookAppService.cs*
```csharp
using Abp.Application.Services.Dto;
using Abp.Domain.Repositories;
using Abp.Extensions;
using Abp.Linq.Extensions;
using BookTracking.Application.Books.Dto;
using BookTracking.Core.Books;
using System.Linq.Dynamic.Core;
using Abp.Application.Services;
using Abp.UI;
using Microsoft.EntityFrameworkCore;


namespace BookTracking.Application.Books;

public class BookAppService(IRepository<Book, int> bookRepository) : ApplicationService, IBookAppService
{
    public async Task<PagedResultDto<BookListDto>> GetBooks(GetBooksInput input)
    {
        var booksQuery = await bookRepository.GetAllAsync();

        var totalCount = await booksQuery.CountAsync();

        var pagedAndFilteredBooks = await booksQuery
            .OrderBy(input.Sorting)
            .PageBy(input)
            .ToListAsync();
        
        var bookList = ObjectMapper.Map<List<BookListDto>>(pagedAndFilteredBooks);

        return new PagedResultDto<BookListDto>(totalCount, bookList);
    }

    public async Task<CreateOrEditBookDto> GetUserForEdit(int id)
    {
        var book = await bookRepository.GetAsync(id);
        
        return ObjectMapper.Map<CreateOrEditBookDto>(book);
    }

    public async Task CreateOrEditBook(CreateOrEditBookDto input)
    {
        if (input.Id.HasValue)
        {
            await UpdateBook(input);
        }
        else
        {
            await CreateBook(input);
        }
    }

    public async Task DeleteBook(int id)
    {
        var book = await bookRepository.GetAsync(id);
        
        if (book == null)
        {
            throw new UserFriendlyException(L("BookNotFound"));
        }
        
        await bookRepository.DeleteAsync(book);
    }

    private async Task CreateBook(CreateOrEditBookDto input)
    {
        var book = ObjectMapper.Map<Book>(input);
        
        await bookRepository.InsertAsync(book);
    }

    private async Task UpdateBook(CreateOrEditBookDto input)
    {
        var book = await bookRepository.GetAsync(input.Id!.Value);
        
        ObjectMapper.Map(input, book);
        
        await bookRepository.UpdateAsync(book);
    }
}
```

#### AutoMapper Configuration

Add your dto mapper to the `BookTracking.Application` project: 

*BookTrackingDtoMapper.cs*
```csharp
using AutoMapper;
using BookTracking.Application.Books.Dto;
using BookTracking.Core.Books;

namespace BookTracking.Application
{
    internal static class BookTrackingDtoMapper
    {
        public static void CreateMappings(IMapperConfigurationExpression configuration)
        {
            configuration.CreateMap<Book, BookListDto>().ReverseMap();
            configuration.CreateMap<Book, CreateOrEditBookDto>().ReverseMap();
            
        }
    }
}
```

#### Application Module

*BookTrackingApplicationModule.cs*
```csharp
using Abp.AutoMapper;
using Abp.Modules;
using Abp.Reflection.Extensions;
using BookTracking.Core.Authorization;
using BookTracking.Core;

namespace BookTracking.Application
{
    /// <summary>
    /// Application layer module of the application.
    /// </summary>
    [DependsOn(typeof(BookTrackingCoreModule))]
    public class BookTrackingApplicationModule : AbpModule
    {
        public override void PreInitialize()
        {
            //Adding authorization providers
            Configuration.Authorization.Providers.Add<BookTrackingAuthorizationProvider>();

            //Adding custom AutoMapper configuration
            Configuration.Modules.AbpAutoMapper().Configurators.Add(BookTrackingDtoMapper.CreateMappings);
        }

        public override void Initialize()
        {
            IocManager.RegisterAssemblyByConvention(typeof(BookTrackingApplicationModule).GetAssembly());
        }
    }
}
```

### 3. Add MVC Project:
Finally, add a new aspnetcore mvc project named `BookTracking.Mvc` in the same `BookTracking` folder. This project will base for the MVC controllers and views of `BookTracking` module.

* Add reference to the `BookTracking.Application` project in the `BookTracking.Mvc` project.

* Add following nuget packages to the `BookTracking.Mvc` project:

```xml
<ItemGroup>
    <PackageReference Include="Abp.AspNetZeroCore.Web" Version="5.0.0" />
</ItemGroup>
```

#### Navigation Provider

Add following class to the `BookTracking.Mvc` project:

*BookTrackingWebNavigationProvider.cs*
```csharp
using Abp.Application.Navigation;
using Abp.Authorization;
using Abp.Localization;
using BookTracking.Core.Authorization;

namespace BookTracking.Mvc;

public class BookTrackingWebNavigationProvider : NavigationProvider
{
    private const string MenuName = "App";

    public override void SetNavigation(INavigationProviderContext context)
    {
        var menu = context.Manager.Menus[MenuName] = new MenuDefinition(MenuName, new FixedLocalizableString("Main Menu"));
        
        menu.AddItem(new MenuItemDefinition(
                    "Book",
                    L("Book"),
                    url: "Books",
                    icon: "fa fa-book",
                    permissionDependency: new SimplePermissionDependency(BookTrackingPermissions.BookTracking_Book)
                )
            );
    }

    private static ILocalizableString L(string name)
    {
        return new LocalizableString(name, "BookTracking");
    }
}
```

#### Embedded Resources

Add following class to the `BookTracking.Mvc` project:

*BookTrackingEmbedResourcesConfigurer.cs*
```csharp
using System.Reflection;
using Abp.Resources.Embedded;

namespace BookTracking.Mvc;

public static class BookTrackingEmbedResourcesConfigurer
{
    public static void Configure(IEmbeddedResourcesConfiguration configuration)
    {
        configuration.Sources.Add(
            new EmbeddedResourceSet(
                "/Resources/",
                Assembly.GetExecutingAssembly(),
                "BookTracking.Mvc.Resources"
            )
        );
    }
}
```

#### Models

Create a sub folder named `Books` inside `Models` in the `BookTracking.Mvc` project. Add a new class named `CreateOrEditBookModalViewModel.cs`.

*CreateOrEditBookModalViewModel.cs*
```csharp
using BookTracking.Application.Books.Dto;

namespace BookTracking.Mvc.Models.Books;

public class CreateOrEditBookModalViewModel
{
    public CreateOrEditBookDto Book { get; set; }
}
```

#### Views

* Create a new folder named `Books` in the `Views` folder of the `BookTracking.Mvc` project.

* Create following class in the `Views` folder

`BookTrackingRazorPage.cs`
```csharp
using Abp.AspNetCore.Mvc.Views;
using Abp.Runtime.Session;
using BookTracking.Core;
using Microsoft.AspNetCore.Mvc.Razor.Internal;

namespace BookTracking.Mvc.Views
{
    public abstract class BookTrackingRazorPage<TModel> : AbpRazorPage<TModel>
    {
        [RazorInject] public IAbpSession AbpSession { get; set; }
        
        protected BookTrackingRazorPage()
        {
            LocalizationSourceName = BookTrackingConsts.LocalizationSourceName;
        }
    }
}
```

* Update `_ViewImports.cshtml` file in the `Views` folder of the `BookTracking.Mvc` project.

```csharp
@using Abp.Localization
@inherits BookTracking.Mvc.Views.BookTrackingRazorPage<TModel>
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, BookTracking.Mvc.Web
```

* Update `_ViewStart.cshtml` file in the `Views` folder of the `BookTracking.Mvc` project.

```csharp
@using Abp.Configuration
@inject SettingManager SettingManager
@{
    var theme = await SettingManager.GetSettingValueAsync("App.UiManagement.Theme");
    Layout = "~/Areas/App/Views/Layout/" + theme + "/_Layout.cshtml";
}
```

Add a new view named `Index.cshtml` in the `Books` folder.

*Index.cshtml*
```html
@using BookTracking.Core.Authorization

@{
    ViewData["Title"] = "Home Page";
    ViewBag.CurrentPageName = "Books";
}

@section Scripts
{
    <script src="~/Resources/Books/Index.js" asp-append-version="true"></script>
}

<div class="app-toolbar py-3 py-lg-6" id="kt_app_toolbar">
    <div id="kt_app_toolbar_container" class="container-xxl app-container d-flex flex-stack ">
        <!--begin::Info-->
        <div class="page-title d-flex flex-column justify-content-center flex-wrap me-3">
            <!--begin::Page Title-->
            <h1 class="page-heading d-flex text-dark fw-bold fs-3 flex-column justify-content-center my-0">
                @L("Books")
            </h1>
            <!--end::Page Title-->
            <span class="text-muted me-4">@L("BooksHeaderInfo")</span>

        </div>
        <!--end::Info-->

        <!--begin::Toolbar-->
        <div class="d-flex align-items-center">
            @if (IsGranted(BookTrackingPermissions.BookTracking_Book_Create))
            {
                <button id="CreateNewBookButton" class="btn btn-primary">
                    <i class="fa fa-plus btn-md-icon"></i>
                    <span class="d-none d-md-inline-block">
                        @L("CreateNewBook")
                    </span>
                </button>
            }
        </div>
        <!--end::Toolbar-->
    </div>
</div>

<div class="app-container container-xxl">
    <div class="card">
        <div class="card-body">
            <div class="align-items-center">
                <table id="BooksTable" class="table align-middle table-row-dashed fs-6 gy-5 dataTable dtr-inline no-footer" width="100%">
                    <thead>
                    <tr>
                        <th></th>
                        <th>@L("Actions")</th>
                        <th>@L("Name")</th>
                        <th>@L("Description")</th>
                        <th>@L("Author")</th>
                        <th>@L("ISBN")</th>
                        <th>@L("TotalPages")</th>
                    </tr>
                    </thead>
                </table>
            </div>
        </div>
    </div>
</div>
```

* Next, add a new view named `_CreateOrEditModal.cshtml` in the `Books` folder.

*_CreateOrEditModal.cshtml*
```html
@using BookTracking.Mvc.Models.Books
@using Microsoft.AspNetCore.Mvc.TagHelpers
@model CreateOrEditBookModalViewModel

<div class="modal-header">
    <h5 class="modal-title">
        @if (Model.IsEditMode)
        {
            <span>@L("EditBookHeader", Model.Book.Name)</span>
        }
        else
        {
            <span>@L("CreateNewBookHeader")</span>
        }
    </h5>
    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-hidden="true"></button>
</div>


<div class="modal-body">
    <form name="CreateOrEditBookForm" role="form" novalidate class="form-validation">
        @if (Model.IsEditMode)
        {
            <input type="hidden" name="Id" value="@Model.Book.Id"/>
        }
        
        <div class="mb-5">
            <label for="Name" class="form-label required">@L("Name")</label>
            <input id="Name" class="form-control" type="text" name="Name" required value="@Model.Book.Name">
        </div>
        <div class="mb-5">
            <label for="Author" class="form-label required">@L("Author")</label>
            <input id="Author" class="form-control" type="text" name="Author" required value="@Model.Book.Author">
        </div>
        <div class="mb-5">
            <label for="Description" class="form-label required">@L("Description")</label>
            <input id="Description" class="form-control" type="text" name="Description" required value="@Model.Book.Description">
        </div>
        <div class="mb-5">
            <label for="ISBN" class="form-label required">@L("ISBN")</label>
            <input id="ISBN" class="form-control" type="text" name="ISBN" required value="@Model.Book.ISBN">
        </div>
        <div class="mb-5">
            <label for="TotalPages" class="form-label required">@L("TotalPages")</label>
            <input id="TotalPages" class="form-control" type="text" name="TotalPages" required value="@Model.Book.TotalPages">
        </div>
    </form>
</div>

<div class="modal-footer">
    <button type="button" class="btn btn-light-primary fw-bold close-button" data-bs-dismiss="modal">@L("Cancel")</button>
    <button type="button" class="btn btn-primary save-button"><i class="fa fa-save"></i> <span>@L("Save")</span></button>
</div>
```

#### Controller

Let's add a controller to the `BookTracking.Mvc` project:

*BooksController.cs*
```csharp
using Abp.Application.Services.Dto;
using Abp.AspNetCore.Mvc.Controllers;
using BookTracking.Application.Books;
using BookTracking.Application.Books.Dto;
using BookTracking.Mvc.Models.Books;
using Microsoft.AspNetCore.Mvc;

namespace BookTracking.Mvc.Controllers;

public class BooksController : AbpController
{
    private readonly IBookAppService _bookAppService;

    public BooksController(IBookAppService bookAppService)
    {
        _bookAppService = bookAppService;
    }
    
    public IActionResult Index()
    {
        return View();
    }
    
    public async Task<IActionResult> CreateOrEditModal(int? id)
    {
        var viewModel = new CreateOrEditBookModalViewModel();
        
        if (id.HasValue)
        {
            viewModel.Book = await _bookAppService.GetUserForEdit(id.Value);
        } else {
            viewModel.Book = new CreateOrEditBookDto();
        }
        
        return PartialView("_CreateOrEditModal", viewModel);
    }
}
```

#### CSS and JS

> For this application, i just need javascript files. If you need css files, you can add them in the same way. 

First configure csproj file of the `BookTracking.Mvc` project to include the embedded files. Add the following code to the `BookTracking.Mvc.csproj` file:

```xml
<ItemGroup>
    <EmbeddedResource Include="Views\**\*.*"/>
    <EmbeddedResource Include="Resources\**\*.*"/>
</ItemGroup>
```

Add js files to the `Views/Books` folder.

*_CreateOrEditModal.js*
```javascript
(function ($) {
    app.modals.CreateOrEditBookModal = function () {
        var _modalManager;
        var _bookService = abp.services.app.book;
        var _$bookForm = null;

        this.init = function (modalManager) {
            _modalManager = modalManager;

            //Initialize your modal here...
            _$bookForm = _modalManager.getModal().find('form[name=CreateOrEditBookForm]');
        };

        this.save = function () {
            var book = _$bookForm.serializeFormToObject();

            _modalManager.setBusy(true);
            _bookService
                .createOrEditBook(book)
                .done(function () {
                    abp.notify.info(app.localize('SavedSuccessfully'));
                    _modalManager.close();
                    abp.event.trigger('app.createOrEditBookModalSaved');
                })
                .always(function () {
                    _modalManager.setBusy(false);
                });
        };

    };
})(jQuery);
```

*Index.js*
```javascript
(function () {
    $(function () {
        var _$booksTable = $('#BooksTable');
        var _bookService = abp.services.app.book;
        
        var _permissions = {
            create: abp.auth.hasPermission('BookTracking.Book.Create'),
            edit: abp.auth.hasPermission('BookTracking.Book.Edit'),
            delete: abp.auth.hasPermission('BookTracking.Book.Delete'),
        };

        var _createOrEditModal = new app.ModalManager({
            viewUrl: abp.appPath + 'Books/CreateOrEditModal',
            scriptUrl: abp.appPath + 'Resources/Books/_CreateOrEditModal.js',
            modalClass: 'CreateOrEditBookModal',
        });

        var dataTable = _$booksTable.DataTable({
            paging: true,
            serverSide: true,
            processing: true,
            listAction: {
                ajaxFunction: _bookService.getBooks,
            },
            columnDefs: [
                {
                    orderable: false,
                    render: function () {
                        return '';
                    },
                    targets: 0,
                },
                {
                    width: 120,
                    targets: 1,
                    data: null,
                    orderable: false,
                    autoWidth: false,
                    type: 'html',
                    defaultContent: '',
                    rowAction: {
                        cssClass: 'btn btn-brand dropdown-toggle',
                        text: '<i class="fa fa-cog"></i> ' + app.localize('Actions') + ' <span class="caret"></span>',
                        items: [
                            {
                                text: app.localize('Edit'),
                                visible: function () {
                                    return _permissions.edit;
                                },
                                action: function (data) {
                                    _createOrEditModal.open({ id: data.record.id });
                                },
                            },
                            {
                                text: app.localize('Delete'),
                                visible: function () {
                                    return _permissions.delete;
                                },
                                action: function (data) {
                                    deleteBook(data.record);
                                },
                            },
                        ],
                    },
                },
                {
                    targets: 2,
                    width: 200,
                    data: 'name'
                },
                {
                    targets: 4,
                    data: 'author'
                },
                {
                    targets: 3,
                    width: 350,
                    data: 'description'
                },
                {
                    targets: 5,
                    data: 'isbn'
                },
                {
                    targets: 6,
                    data: 'totalPages'
                },
            ],
        });

        function getBooks() {
            dataTable.ajax.reload();
        }

        function deleteBook(book) {
            abp.message.confirm(
                app.localize('BookDeleteWarningMessage', book.name),
                app.localize('AreYouSure'),
                function (isConfirmed) {
                    if (isConfirmed) {
                        _bookService
                            .deleteBook(book.id)
                            .done(function () {
                                getBooks();
                                abp.notify.success(app.localize('SuccessfullyDeleted'));
                            });
                    }
                }
            );
        }
        
        $('#CreateNewBookButton').click(function () {
            _createOrEditModal.open();
        });

        abp.event.on('app.createOrEditBookModalSaved', function () {
            getBooks();
        });
    });
})();
```

#### MVC Module

*BookTrackingWebModule.cs*
```csharp
using System.Reflection;
using Abp.AspNetCore.Configuration;
using Abp.AspNetZeroCore.Web;
using Abp.Modules;
using BookTracking.Application;

namespace BookTracking.Mvc;

[DependsOn(
    typeof(BookTrackingApplicationModule),
    typeof(AbpAspNetZeroCoreWebModule)
)]
public class BookTrackingMvcModule : AbpModule
{
    public override void PreInitialize()
    {
        Configuration.Navigation.Providers.Add<BookTrackingWebNavigationProvider>();
        
        Configuration.Modules.AbpAspNetCore()
            .CreateControllersForAppServices(
                typeof(BookTrackingApplicationModule).Assembly, 
                moduleName: "app", 
                useConventionalHttpVerbs: true
            );

        BookTrackingEmbedResourcesConfigurer.Configure(Configuration.EmbeddedResources);
    }

    public override void Initialize()
    {
        IocManager.RegisterAssemblyByConvention(Assembly.GetExecutingAssembly());
    }
}
```

### 4. Integration into the project

In this section, we will integrate the `BookTracking` module into the main project.

#### Add Project References

##### Application Project

* Add reference to the `BookTracking.Application` project in the `LibraryAppDemo.Application` project.

* Configure `LibraryAppDemoApplicationModule.cs` file in the `LibraryAppDemo.Application` project.

```csharp
[DependsOn(
    typeof(LibraryAppDemoApplicationSharedModule),
    typeof(LibraryAppDemoCoreModule),
    typeof(BookTrackingApplicationModule)
    )]
```

##### Core Project

* Add reference to the `BookTracking.Core` project in the `LibraryAppDemo.Core` project.

* Configure `LibraryAppDemoCoreModule.cs` file in the `LibraryAppDemo.Core` project.

```csharp
[DependsOn(
    typeof(LibraryAppDemoCoreSharedModule),
    typeof(AbpZeroCoreModule),
    typeof(AbpZeroLdapModule),
    typeof(AbpAutoMapperModule),
    typeof(AbpAspNetZeroCoreModule),
    typeof(AbpMailKitModule),
    typeof(AbpZeroCoreOpenIddictModule),
    typeof(BookTrackingCoreModule)
    )
]
```

##### EntityFrameworkCore Project

* Add reference to the `BookTracking.Core` project in the `LibraryAppDemo.EntityFrameworkCore` project.

* Configure `LibraryAppDemoEntityFrameworkCoreModule .cs` file in the `LibraryAppDemo.Core` project.

```csharp
[DependsOn(
    typeof(AbpZeroCoreEntityFrameworkCoreModule),
    typeof(LibraryAppDemoCoreModule),
    typeof(AbpZeroCoreOpenIddictEntityFrameworkCoreModule),
    typeof(BookTrackingEntityFrameworkCoreModule)
)]
```

Open the `LibraryAppDemoDbContext` class in the `src/LibraryAppDemo.EntityFrameworkCore` project and add the following code:

```csharp
// BookTracking
// For repository injection
public virtual DbSet<Book> Books { get; set; }
```

* Run the `Add-Migration` command in the Package Manager Console to create a migration for the `BookTracking` module.

* Run the `Update-Database` command in the Package Manager Console to apply the migration to the database.

##### MVC Project

* Add reference to the `BookTracking.Mvc` project in the `LibraryAppDemo.Web.Mvc` project.

* Configure `LibraryAppDemoWebMvcModule.cs` file in the `LibraryAppDemo.Web.Mvc` project.

```csharp
[DependsOn(
    typeof(LibraryAppDemoWebCoreModule),
    typeof(BookTrackingMvcModule)
)]
```

* Update `AppNavigationProvider.cs` file in the `LibraryAppDemo.Web.Mvc` project.

```csharp
public override void SetNavigation(INavigationProviderContext context)
{

    var menu = context.Manager.Menus[MenuName] = context.Manager.Menus.ContainsKey(MenuName)
        ? context.Manager.Menus[MenuName]
        : new MenuDefinition(MenuName, new FixedLocalizableString("Main Menu"));

// other codes
}
```

* Add following code line to the `Startup.cs` file in the `LibraryAppDemo.Web.Mvc` project.

```csharp
app.UseStaticFiles();
// After statices
app.UseEmbeddedFiles();
```
Here is the final product

![Final Product](images/Blog/modular-monolith-application-result.png)

## Source Code

You can find the source code of the example project on [GitHub](https://github.com/aspnetzero/aspnet-zero-samples/tremastere/master/LibraryAppDemo)

## Conclusion

* In this blog post, we have explored how to build a modular monolith application using ASP.NET Zero. We have created a `BookTracking` module with the `BookTracking.Core`, `BookTracking.Application`, and `BookTracking.Mvc` projects. We have integrated the `BookTracking` module into the main project by adding project references and configuring the modules. 

* By following these steps, you can create a modular monolith application with ASP.NET Zero and take advantage of the simplicity and modularity of this architecture.