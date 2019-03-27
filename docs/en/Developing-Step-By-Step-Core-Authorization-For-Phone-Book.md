# Authorization For Phone Book

At this point, anyone can enter phone book page since no authorization
defined. We will define two permission:

- A permission to **enter phone book page**.
- A permission to **create new person** (which is a child permission
  of first one, as naturally).

## Permission for Entering Phone Book Page

### Define the permission

Go to **AppAuthorizationProvider** class and add a new permission as shown below:

```csharp
pages.CreateChildPermission(AppPermissions.Pages_Tenant_PhoneBook, L("PhoneBook"), multiTenancySides: MultiTenancySides.Tenant);
```

A permission should have a unique name. We define permission names as constant strings in **AppPermissions** class. It's a simple constant string:

```csharp
public const string Pages_Tenant_PhoneBook = "Pages.Tenant.PhoneBook";
```

Unique name of this permission is "**Pages.Tenant.PhoneBook**". While you can set any string (as long as it's unique), it's suggested to use this convention. A permission can have a localizable display name: "**PhoneBook**" here. (See "Adding a New Page" section for more about localization, since it's very similar). Lastly, we set this as a **tenant** level permission.

### Add AbpAuthorize attribute

**AbpAuthorize** attribute can be used as **class level** or **method level** to protect an application service or service method from unauthorized users. Since all server side code is located in PersonAppService class, we can declare a class level attribute as shown below:

```csharp
[AbpAuthorize(AppPermissions.Pages_Tenant_PhoneBook)]
public class PersonAppService : PhoneBookAppServiceBase, IPersonAppService
{
    //...
}
```

Now, let's try to enter Phone Book page by clicking the menu item:

<img src="images/error-500.png" alt="500 Error" class="img-thumbnail" width="614" height="243" />

We get an error message. This exception is thrown when any method of PersonAppService is called without required permission.

### Hide Unauthorized Menu Item

This secures the service, but we should also **hide** the Phone book **menu item**. It's easy, open **AppNavigationProvider** and add requiredPermissionName as shown below:

```csharp
new MenuItemDefinition(
    PageNames.App.Tenant.PhoneBook,
    L("PhoneBook"),
    url: "tenant.phonebook",
    icon: "glyphicon glyphicon-book",
    requiredPermissionName: AppPermissions.Pages_Tenant_PhoneBook
)
```

### Grant permission

So, how we can enter the page now? Simple, go to **Role Management** page and edit **admin** role:

<img src="images/role-permissions-with-phonebook2.png" alt="Role permissions" class="img-thumbnail" width="839" height="898" />

We see that a **new permission** named "**Phone book**" added to **permissions** tab. So, we can check it and save the role. After saving, we need to **refresh** the whole page to refresh permissions for the current user. We could also grant this permission for a specific user. Now, we can enter the Phone book page again.

## Permission for Create New Person

While a permission for a page is useful and probably needed always, we
may need to define additional permissions to perform some **specific
actions** on a page, like creating a new person.

### Define the Permission

Defining a permission is similar:

```csharp
var phoneBook = pages.CreateChildPermission(AppPermissions.Pages_Tenant_PhoneBook, L("PhoneBook"), multiTenancySides: MultiTenancySides.Tenant);
phoneBook.CreateChildPermission(AppPermissions.Pages_Tenant_PhoneBook_CreatePerson, L("CreateNewPerson"), multiTenancySides: MultiTenancySides.Tenant);
```

First permission was defined before. In the second line, we are creating a child permission of first one.

### Add AbpAuthorize Attribute

This time, we're declaring **AbpAuthorize** attribute just for **CreatePerson** method:

```csharp
[AbpAuthorize(AppPermissions.Pages_Tenant_PhoneBook_CreatePerson)]
public async Task CreatePerson(CreatePersonInput input)
{
    //...
}
```

### Hide Unauthorized Button

If we run the application and try to create a person, we get an authorization error after clicking the save button. But, it's good to **completely hide Create New Person button** if we don't have the permission. It's very simple: 

Open the **index.cshtml** razor view and use **IsGranted** method:

```html
@if (IsGranted(AppPermissions.Pages_Tenant_PhoneBook_CreatePerson))
{
    <button id="CreateNewPersonButton" class="btn btn-primary blue"><i class="fa fa-plus"></i> @L("CreateNewPerson")</button>
}
```

In this way, the "Create New Person" button does not rendered in server and user can not see this button.

### Grant permission

To see the button again, we can go to role or user manager and grant related permission as shown below:

<img src="images/user-permissions-phonebook2.png" alt="User specific permissions" class="img-thumbnail" />

As shown above, **Create new person** permission is a child permission of the **Phone book**.

## Authorization for MVC Controllers

We added Authorize attributes to PersonAppService. This prevents unauthorized calls to this service. But unauthorized clients **still can call** actions of PhoneBookController actions. Since PhoneBookController uses PersonAppService, they will get authorization exception and can not use the service. But we can also secure MVC Controllers. This is suggested since some MVC actions may not use application services. Also,
it's better to prevent unauthorized attempts at the very beginning.

We use **AbpMvcAuthorize** attribute for MVC Controllers as shown below:

```csharp
[AbpMvcAuthorize(AppPermissions.Pages_Tenant_PhoneBook)]
public class PhoneBookController : PhoneBookControllerBase
{
    ...

    [AbpMvcAuthorize(AppPermissions.Pages_Tenant_PhoneBook_CreatePerson)]
    public PartialViewResult CreatePersonModal()
    {
        ...
    }

    ...
}
```

See [authorization documentation](https://aspnetboilerplate.com/Pages/Documents/Authorization) for more information on authorization.

## Next

- [Deleting a Person](Developing-Step-By-Step-Core-Deleting-Person.md)
