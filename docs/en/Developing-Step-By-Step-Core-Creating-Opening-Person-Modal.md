# Opening the Person Modal

We need to put a "Create new person" button to the 'people list page'
and write some javascript code to open the modal when clicked to the
button.

So, changing the **Index.cshtml** view header as shown below:

```html
@using System.Threading.Tasks
@using Acme.PhoneBook.Web.Areas.App.Startup
@model Acme.PhoneBook.Web.Areas.App.Models.PhoneBook.IndexViewModel

@{
    ViewBag.CurrentPageName = AppPageNames.Tenant.PhoneBook;
}

@section Scripts
{
    <environment names="Development">
        <script src="~/view-resources/Areas/App/Views/PhoneBook/_CreatePersonModal.js" asp-append-version="true"></script>
        <script src="~/view-resources/Areas/App/Views/PhoneBook/Index.js" asp-append-version="true"></script>
    </environment>
}

<div class="row kt-margin-b-5">
    <div class="col-xs-6">
        <div class="page-head">
            <div class="page-title">
                <h1>
                    <span>@L("PhoneBook")</span>
                </h1>
            </div>
        </div>
    </div>

    <div class="col-xs-6 text-right">
        <button id="CreateNewPersonButton" class="btn btn-primary"><i class="fa fa-plus"></i> @L("CreateNewPerson")</button>
    </div>
</div>

<div class="portlet light">
    <div class="portlet-body">

        <h3>@L("AllPeople")</h3>

        <div class="list-group">
            @foreach (var person in Model.Items)
            {
                <a href="javascript:;" class="list-group-item">
                    <h4 class="list-group-item-heading">
                        @person.Name @person.Surname
                    </h4>
                    <p class="list-group-item-text">
                        @person.EmailAddress
                    </p>
                </a>
            }
        </div>

    </div>
</div>
```

We included modal's javascript (\_CreatePersonModal.js) and a
**Index.js** file which is defined as shown below:

```javascript
(function () {
    var _createPersonModal = new app.ModalManager({
        viewUrl: abp.appPath + 'App/PhoneBook/CreatePersonModal',
        scriptUrl: abp.appPath + 'view-resources/Areas/App/Views/PhoneBook/_CreatePersonModal.js',
        modalClass: 'CreatePersonModal'
    });

    $('#CreateNewPersonButton').click(function (e) {
        e.preventDefault();
        _createPersonModal.open();
    });
})();
```

**ModalManager** is a predefined modal helper javascript class of AspNet
Zero. It accepts a **viewUrl** (which is actually an MVC action to load
the view), a **scriptUrl** (javascript file of the modal) and a
**modalClass** (which is set when we define the modal above).

ModalManager's **open** method loads view and javascript (if needed) and
opens the modal.

Lastly, we should create the MVC action defined as
App/PhoneBook/CreatePersonModal. So, open the **PhoneBookController**
again and add the following action:

```csharp
public PartialViewResult CreatePersonModal()
{
    return PartialView("_CreatePersonModal");
}
```

Now, we can run the application and open the modal by clicking the
'Create New Person' button:

<img src="images/phonebook-create-person-dialog2.png" alt="Create Person Dialog" class="img-thumbnail" />

## Next

- [Saving the Person](Developing-Step-By-Step-Core-Saving-Person.md)