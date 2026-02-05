# Edit Mode For People

Now we want to edit name, surname and e-mail of people:

<img src="images/edit-person-core.png" alt="Edit Person" class="img-thumbnail" width="548" height="333" />  

First of all, we create the necessary DTOs to transfer people's id, name,
surname and e-mail. Then create the functions in PersonAppService for
editing people:  

First, create a mapper for the edit operations. Create `PersonToGetPersonForEditOutputMapper.cs`:

```csharp
using Riok.Mapperly.Abstractions;

namespace Acme.PhoneBookDemo.PhoneBook.Mapper;

[Mapper]
public partial class PersonToGetPersonForEditOutputMapper
{
    public partial GetPersonForEditOutput Map(Person person);
}
```

Then implement the methods using ObjectMapper:

```csharp
public async Task<GetPersonForEditOutput> GetPersonForEdit(IEntityDto input)
{
    var person = await _personRepository.GetAsync(input.Id);
    return ObjectMapper.Map<GetPersonForEditOutput>(person);
}

public async Task EditPerson(EditPersonInput input)
{
    var person = await _personRepository.GetAsync(input.Id);
    person.Name = input.Name;
    person.Surname = input.Surname;
    person.EmailAddress = input.EmailAddress;
    await _personRepository.UpdateAsync(person);
}
```

## View

```html
@using Acme.PhoneBook.People
@using Acme.PhoneBook.Web.Areas.App.Models.Common.Modals
@using Acme.PhoneBook.Web.Areas.App.Models.PhoneBook
@Html.Partial("~/Areas/App/Views/Common/Modals/_ModalHeader.cshtml", new ModalHeaderViewModel("Edit Person"))
@model EditPersonModalViewModel

<div class="modal-body">
    <form role="form" novalidate class="form-validation">

        <input type="hidden" name="Id" value="@Model.Id" />

        <div class="form-group form-md-line-input  no-hint">
            <input class="form-control" type="text" name="Name" value="@Model.Name" required maxlength="@PersonConsts.MaxNameLength">
            <label>@L("Name")</label>
        </div>
        <div class="form-group form-md-line-input  no-hint">
            <input type="text" name="Surname" value="@Model.Surname" class="form-control" required maxlength="@PersonConsts.MaxSurnameLength">
            <label>@L("Surname")</label>
        </div>

        <div class="form-group form-md-line-input  no-hint">
            <input type="email" name="EmailAddress" value="@Model.EmailAddress" class="form-control"  maxlength="@PersonConsts.MaxEmailAddressLength">
            <label>@L("EmailAddress")</label>
        </div>
    </form>
</div>

@Html.Partial("~/Areas/App/Views/Common/Modals/_ModalFooterWithSaveAndCancel.cshtml")
```

## View Model

```csharp
using Acme.PhoneBookDemo.PhoneBook;

namespace Acme.PhoneBook.Web.Areas.App.Models.PhoneBook
{
    public class EditPersonModalViewModel : GetPersonForEditOutput
    {

    }
}
```

Create a mapper for the view model. Create `GetPersonForEditOutputToEditPersonModalViewModelMapper.cs`:

```csharp
using Riok.Mapperly.Abstractions;

namespace Acme.PhoneBook.Web.Areas.App.Models.PhoneBook;

[Mapper]
public partial class GetPersonForEditOutputToEditPersonModalViewModelMapper
{
    public partial EditPersonModalViewModel Map(GetPersonForEditOutput output);
}
```

## Scripts

Create **\_editPersonModal.js**:

```javascript
(function ($) {
        app.modals.EditPersonModal = function () {

        var _modalManager;
        var _personService = abp.services.app.person;
        var _$form = null;

        this.init = function (modalManager) {
            _modalManager = modalManager;
            personId = _modalManager.getArgs().personId;
            _$form = _modalManager.getModal().find('form');
            _$form.validate();
        };

        this.save = function () {
            if (!_$form.valid()) {
                return;
            }

            var person = _$form.serializeFormToObject();

            _modalManager.setBusy(true);
            _personService.editPerson(person).done(function () {
                _modalManager.close();
                location.reload();
            }).always(function () {
                _modalManager.setBusy(false);
            });
        };
    };
})(jQuery);
```

Add Those lines to **index.js**:

```javascript
    var _editPersonModal = new app.ModalManager({
        viewUrl: abp.appPath + 'App/PhoneBook/EditPersonModal',
        scriptUrl: abp.appPath + 'view-resources/Areas/App/Views/PhoneBook/_EditPersonModal.js',
        modalClass: 'EditPersonModal'
    });

    $('#AllPeopleList button.edit-person').click(function (e) {
        e.preventDefault();
        var $listItem = $(this).closest('.list-group-item');
        var id = $listItem.data('person-id');
        _editPersonModal.open({ id: id });
    });
```

Add this method to **PhoneBookController.cs**:

```csharp
public async Task<PartialViewResult> EditPersonModal(int id)
{
    var output = await _personAppService.GetPersonForEdit(new EntityDto { Id = id });
    var viewModel = ObjectMapper.Map<EditPersonModalViewModel>(output);

    return PartialView("_EditPersonModal", viewModel);
}
```

## Next

- [Multi Tenancy](Developing-Step-By-Step-Core-Multi-Tenancy.md)