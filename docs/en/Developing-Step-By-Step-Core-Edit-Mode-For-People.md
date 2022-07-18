# Edit Mode For People

Now we want to edit name, surname and e-mail of people. Here is the final result of the edit person modal:

<img src="images/edit-person-core-2.png" alt="Edit Person" class="img-thumbnail" width="548" height="333" />  

Now go to **PersonAppservice** and **IPersonAppService** add **EditPerson**.

```csharp       
public class EditPersonInput : EntityDto
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
```csharp
public async Task EditPerson(EditPersonInput input)
{
    var person = await _personRepository.GetAsync(input.Id);
    person.Name = input.Name;
    person.Surname = input.Surname;
    person.EmailAddress = input.EmailAddress;
    await _personRepository.UpdateAsync(person);
}
```
Then fill person edit tab in the **_EditPersonModal.cshtml**
```html
<div class="tab-content">
<!--Fill the Person Tab-->
    <div class="tab-pane pt-5 active" id="Person" role="tabpanel">
            <form id="editPersonForm">
                <input type="hidden" name="Id" value="@Model.Id">
                <div class="my-3">
                    <label for="name" class="form-label">@L("Name")</label>
                    <input id="name" class="form-control" type="text" name="Name" value="@Model.Name" required  maxlength="@PersonConsts.MaxNameLength">
                </div>
                <div class="my-3">
                    <label for="surname" class="form-label">@L("Surname")</label>
                    <input id="surname" class="form-control" type="text" name="Surname" value="@Model.Surname" required  maxlength="@PersonConsts.MaxSurnameLength">
                </div>
                
                <div class="my-3">
                    <label for="emailAddress" class="form-label">@L("EmailAddress")</label>
                    <input id="emailAddress" class="form-control" type="text" name="EmailAddress" value="@Model.EmailAddress" required  maxlength="@PersonConsts.MaxEmailAddressLength">
                </div>
                
                <div class="my-3">
                    <button class="btn btn-primary" id="editPerson" type="button">@L("Save")</button>
                </div>
            </form>
    </div>
<!--Fill the Person Tab-->
```

## Scripts

Add related code part to **_EditPersonModal.js**. Here is the final result of the **_EditPersonModal.js**

```javascript
(function ($) {
    app.modals.EditPersonModal = function () {
        var _modalManager;
        var _personService = abp.services.app.person;
        var _$addPhoneForm = null;
        var _$editPersonForm = null;
        var _$phoneNumbersTable = null;

        var hasChanges = false;

        function getPhoneTypeString(phoneType) {
            switch (phoneType) {
                case "1":
                    return app.localize('Mobile');
                case "2":
                    return app.localize('Home');
                default:
                    return app.localize('Business');
            }
        }

        function addPhoneToPhoneNumbersTable(phone) {
            var row = `<tr id="phoneNumberRow-${phone.PersonId}">
                        <td>${getPhoneTypeString(phone.Type)}</td>
                        <td>${phone.Number}</td>
                         <td style="width:100px;">
                            <button class="btn btn-danger btn-delete-phone btn-sm" data-phone-id="${phone.id}">
                                <i class="la la-floppy-o"></i>
                                ${app.localize('Delete')}
                            </button>
                        </td>
                    </tr>`;
            _$phoneNumbersTable.find('tbody').append(row);
        }

        function addPhone() {
            if (!_$addPhoneForm.valid()) {
                return;
            }

            var phone = _$addPhoneForm.serializeFormToObject();


            _modalManager.setBusy(true);
            _personService.addPhone(phone).done(function () {
                addPhoneToPhoneNumbersTable(phone);
                abp.notify.info(app.localize('SavedSuccessfully'));
                hasChanges = true;
            }).always(function () {
                _modalManager.setBusy(false);
            });
        }

        function deletePhone(phoneId) {
            if (!phoneId) {
                return;
            }
            
            abp.message.confirm(
                app.localize('DeletePhoneWarningMessage'),
                app.localize('AreYouSure'),
                function (isConfirmed) {
                    if (isConfirmed) {
                        _modalManager.setBusy(true);
                        _personService.deletePhone({id: phoneId}).done(function () {
                            $('#phoneNumberRow-' + phoneId).remove();
                            abp.notify.info(app.localize('SavedSuccessfully'));
                            hasChanges = true;
                        }).always(function () {
                            _modalManager.setBusy(false);
                        });
                    }
                }
            );            
        }

        function saveEditPerson() {
            if (!_$editPersonForm.valid()) {
                return;
            }

            var person = _$editPersonForm.serializeFormToObject();

            _modalManager.setBusy(true);
            _personService.editPerson(person).done(function () {
                abp.notify.info(app.localize('SavedSuccessfully'));
                hasChanges = true;
            }).always(function () {
                _modalManager.setBusy(false);
            });
        }

        this.init = function (modalManager) {
            _modalManager = modalManager;

            _$addPhoneForm = _modalManager.getModal().find('#addPhoneForm');
            _$addPhoneForm.validate();

            _$editPersonForm = _modalManager.getModal().find('#editPersonForm');
            _$editPersonForm.validate();

            _$phoneNumbersTable = _modalManager.getModal().find('#tablePhoneNumbers');

            _$addPhoneForm.find('#BtnAddPhone').click(function (e) {
                addPhone();
            });

            _modalManager.getModal().find('.btn-delete-phone').click(function (e) {
                deletePhone($(this).data('phone-id'));
            });

            _modalManager.getModal().find('.close-button').click(function () {
                if (hasChanges) {
                    abp.event.trigger('app.personEditClosed');
                }
            })

            _$editPersonForm.find('#editPerson').click(function (e) {
                saveEditPerson();
            });
        };

        this.save = function () {

        };
    };
})(jQuery);
```

## Next

- [Multi Tenancy](Developing-Step-By-Step-Core-Multi-Tenancy.md)