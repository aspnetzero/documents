## Deleting a Person

Let's add a delete button in people list as shown below:

<img src="images/phonebook-people-delete-button-4.png" alt="Delete person" class="img-thumbnail" />

We're starting from UI in this case.

### View

We're changing **index.cshtml** view to add a button (related part is
shown here):

```html
<table class="table align-middle table-row-dashed fs-6 gy-5 dataTable no-footer" id="AllPeopleList">
    <thead>
    <tr>
        <th style="width:50px"></th>
        <th>@L("Name")</th>
        <th>@L("Surname")</th>
        <th>@L("EmailAddress")</th>
    </tr>
    </thead>
    <tbody>
   @foreach (var person in Model.Items)
    {
        <tr data-person-id="@person.Id">
            <td>
                <div class="dropdown dropdown-inline ml-2">
                    <button class="btn btn-primary btn-sm dropdown-toggle" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="true">
                        <i class="fa fa-cog"></i> 
                        <span class="d-none d-md-inline-block d-lg-inline-block d-xl-inline-block">@L("Actions")</span> 
                        <span class="caret"></span>
                    </button>
                    
                    <!--begin::Navigation-->
                    <ul class="dropdown-menu dropdown-menu-md dropdown-menu-right" x-placement="bottom-end">
                        @if (IsGranted(AppPermissions.Pages_Tenant_PhoneBook_DeletePerson))
                        {
                            <li>
                                <button class="dropdown-item btn-delete-person text-danger">
                                    @L("Delete")
                                </button>
                            </li>
                        }
                    </ul>
                </div>
            </td>
            <td>@person.Name</td>
            <td>@person.Surname</td>
            <td>@person.EmailAddress</td>
        </tr>
    }
    </tbody>
</table>
```

Surely, we defined 'delete person' permission as like before.

### Javascript

Now, adding code to delete person (to Index.js):

```javascript
var _personService = abp.services.app.person;

$('#AllPeopleList button.btn-delete-person').click(function (e) {
    e.preventDefault();

    var $row = $(this).closest('tr');
    var personId = $row.attr('data-person-id');

    abp.message.confirm(
        app.localize('AreYouSureToDeleteThePerson'),//message
        app.localize('AreYouSure'),//title
        function(isConfirmed) {
            if (isConfirmed) {
                _personService.deletePerson({
                    id: personId
                }).done(function () {
                    abp.notify.info(app.localize('SuccessfullyDeleted'));
                    $row.remove();
                });
            }
        }
    );
});
```

It first shows a confirmation message when we click the delete button:

<img src="images/confirmation-delete-person-3.png" alt="Confirmation message" class="img-thumbnail" />

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

  