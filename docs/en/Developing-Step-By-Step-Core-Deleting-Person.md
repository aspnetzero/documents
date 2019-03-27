## Deleting a Person

Let's add a delete button in people list as shown below:

<img src="images/phonebook-people-delete-button3.png" alt="Delete person" class="img-thumbnail" />

We're starting from UI in this case.

### View

We're changing **index.cshtml** view to add a button (related part is
shown here):

```html
<div id="AllPeopleList" class="list-group">
    @foreach (var person in Model.Items)
    {
        <a href="javascript:;" class="list-group-item" data-person-id="@person.Id">
            <h4 class="list-group-item-heading">
                @person.Name @person.Surname
        @if (IsGranted(AppPermissions.Pages_Tenant_PhoneBook_DeletePerson))
        {
            <button title="@L("Delete")" class="btn btn-circle btn-icon-only btn-danger delete-person" href="javascript:;">
                <i class="la la-trash"></i>
            </button>
        }
            </h4>
            <p class="list-group-item-text">
                @person.EmailAddress
            </p>
        </a>
    }
</div>
```

Surely, we defined 'delete person' permission as like before.

### Style

We're using a **[LESS](http://lesscss.org/)** style here to take button
right. Created a file named **index.less** and added following lines:

```css
#AllPeopleList {
    .list-group-item-heading {
        button.delete-person {
            float: right;
        }
    }
}
```

And add the style to your Index.cshtml page (You can also add minified
versions of styles for other environments like production and staging):

```html
@section Styles
{
    <environment names="Development">
        <link rel="stylesheet" href="~/view-resources/Areas/App/Views/PhoneBook/Index.css" asp-append-version="true" />
    </environment>
}
```

### Javascript

Now, adding code to delete person (to Index.js):

```javascript
var _personService = abp.services.app.person;

$('#AllPeopleList button.delete-person').click(function (e) {
    e.preventDefault();

    var $listItem = $(this).closest('.list-group-item');
    var personId = $listItem.attr('data-person-id');

    abp.message.confirm(
        app.localize('AreYouSureToDeleteThePerson'),
        function(isConfirmed) {
            if (isConfirmed) {
                _personService.deletePerson({
                    id: personId
                }).done(function () {
                    abp.notify.info(app.localize('SuccessfullyDeleted'));
                    $listItem.remove();
                });
            }
        }
    );
});
```

It first shows a confirmation message when we click the delete button:

<img src="images/confirmation-delete-person2.png" alt="Confirmation message" class="img-thumbnail" />

If we click Yes, it simply calls **deletePerson** method of
**PersonAppService** and shows a
**[notification](https://aspnetboilerplate.com/Pages/Documents/Javascript-API/Notification)**
if operation succeed. Also, removes the person from the page using
jQuery's **remove** function.

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

We also **authorized** deleting a person as did before for creating a
person.

We also need to define **Pages\_Tenant\_PhoneBook\_DeletePerson**
constant in AppPermissions and define related permission in
**AppAuthorizationProvider**.

## Next

- [Filtering People](Developing-Step-By-Step-Core-Filtering-People.md)

  