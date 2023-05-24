# Using ASP.NET Zero with DevExtreme Mvc Part 3

Welcome to Part 3 of the tutorial series on using ASP.NET Zero with DevExtreme Mvc. In this tutorial, we will focus on creating a new person in the phone book application by adding a modal for adding a new item, and deleting an item from the phone book.

## Adding CreatePerson Method to PersonAppService

We will start by defining a `CreatePerson` method in the `IPersonAppService` interface. The method will have the following signature:

```csharp
Task CreatePerson(CreatePersonInput input);
```

To support this method, we need to create a corresponding DTO (Data Transfer Object) named `CreatePersonInput`, which will define the parameters for the method. Here is the definition of the `CreatePersonInput` class:

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

We also need to add configuration for AutoMapper in the `CustomDtoMapper.cs` file to map `CreatePersonInput` to the `Person` entity. Additionally, we can remove the corresponding consts from the `Person` entity and define a new `PersonConsts` class in the `.Core.Shared` project to hold the max length constants.

```csharp
configuration.CreateMap<CreatePersonInput, Person>();
```

`CreatePersonInput` is mapped to `Person` entity (comment out related line in `CustomDtoMapper.cs` and we will use mapping below). All properties are decorated with **data annotation attributes** to provide automatic [validation](https://aspnetboilerplate.com/Pages/Documents/Validating-Data-Transfer-Objects). Notice that we use same consts defined in `PersonConsts.cs` in `.Core.Shared` project for `MaxLength` properties. After adding this class, you can remove consts from `Person` entity and use this new consts class.

```csharp
public class PersonConsts
{
    public const int MaxNameLength = 32;
    public const int MaxSurnameLength = 32;
    public const int MaxEmailAddressLength = 255;
}
```

The implementation of the `CreatePerson` method in the `PersonAppService` is straightforward. It involves mapping the input to the Person entity and inserting it into the database using the repository. The method uses the async/await pattern, as recommended in ASP.NET Zero.

```csharp
public async Task CreatePerson(CreatePersonInput input)
{
    var person = ObjectMapper.Map<Person>(input);
    await _personRepository.InsertAsync(person);
}
```

## Testing `CreatePerson` Method

To ensure the correctness of the `CreatePerson` method, we can write unit tests. We will create **two test methods**: one to test the successful creation of a person with valid arguments, and another to test the failure of creating a person with invalid arguments. The tests will utilize the async/await pattern and make use of the `UsingDbContext` helper method provided by the `AppTestBase` class to perform database operations.

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

Test method also written using **async/await** pattern since calling method is async. We called `CreatePerson` method, then checked if given person is in the database. `UsingDbContext` method is a helper method of `AppTestBase` class (which we inherited this unit test class from). It's used to easily get a reference to `DbContext` and use it directly to perform database operations. This method successfully works since all required fields are supplied. Let's try to create a test for **invalid arguments**:

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

We did not set `Surname` property of `CreatePersonInput` despite it being **required**. So, it throws `AbpValidationException` automatically. Also, we can not send null to `CreatePerson` method since validation system also checks it. This test calls CreatePerson with invalid arguments and asserts that it throws `AbpValidationException`. See [validation document](https://aspnetboilerplate.com/Pages/Documents/Validating-Data-Transfer-Objects) for more information.

## Creating New Person

Create a controller action to create a person.

*PhoneBookController.cs* 

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

Then Update `Index.cshml` as seen below:

*Index.cshtml*

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

To enable deleting a person, we add a `DeletePerson` method to the server-side `IPersonAppService` interface. The method takes an `EntityDto` as input, which is a shortcut provided by ASP.NET Zero for passing an ID value. The implementation of the method is straightforward, as it uses the repository to delete the person asynchronously.

### Application Service

Let's add a `DeletePerson` method to the server side. We are adding it to the interface `IPersonAppService`, 

```csharp
Task DeletePerson(EntityDto input);
```

*PersonAppService.cs*

```csharp
[AbpAuthorize(AppPermissions.Pages_Tenant_PhoneBook_DeletePerson)]
public async Task DeletePerson(EntityDto input)
{
    await _personRepository.DeleteAsync(input.Id);
}
```

We also **authorized** deleting a person as did before for creating a person. We also need to define `Pages_Tenant_PhoneBook_DeletePerson` constant in AppPermissions and define related permission in `AppAuthorizationProvider`.

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

We're changing `Index.cshtml` view to add a button;

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

If we click Yes, it simply calls `DeletePerson` method of `PhoneBookController`.

## Conclusion

In this blog post, we covered two important functionalities in the phone book application built with ASP.NET Zero and DevExtreme Mvc: creating a new person and deleting a person.

### Summary

* Creating a new person with ASP.NET Zero and DevExtreme Mvc.
* Adding a `CreatePerson` method to the `PersonAppService`.
* Testing the `CreatePerson` method with valid and invalid arguments.
* Adding the **ability** to create a new person in the **client-side** application.
* Deleting a person from the phone book.
* Implementing the `DeletePerson` method in the `PersonAppService`.
* Updating the client-side service proxies and UI for deleting a person.

### What's Next?

In the upcoming blog post, we will cover several important topics related to our application. We'll start by exploring the process of **editing** a person's information, allowing us to update their **details** as needed. Next, we'll focus on **adding new phone** numbers to a person, enhancing their contact information. We'll also learn how to **delete phone** numbers from a person's record, providing **flexibility** in managing their data. Additionally, we'll guide you through **running the application**, ensuring a smooth execution of the developed features. Finally, we'll **conclude** the blog series, summarizing our journey and achievements. Stay tuned for an informative and engaging read!

See the [next part](https://aspnetzero.com/blog/using-asp.net-zero-with-devextreme-mvc-part-4) of this tutorial.