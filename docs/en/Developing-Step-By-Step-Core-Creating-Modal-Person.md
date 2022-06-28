# Creating Modal for New Person

We will create a Bootstrap Modal to create a new person. First of all,
you can create modal and save as you already know and like. But AspNet
Zero has some helper scripts to make it easier if you use it. We will
use them here.

We can **copy** cshtml and js files from
**Areas/App/Views/Common/Modals/Empty** folder as a base for a new modal
to **Areas/App/Views/PhoneBook/\_CreatePersonModal.cshtml**.

Copied and modified the view code as shown below
(\_CreatePersonModal.cshtml): 

```html
@using Acme.PhoneBookDemo.PhoneBook
@using Acme.PhoneBookDemo.Web.Areas.App.Models.Common.Modals

@Html.Partial("~/Areas/App/Views/Common/Modals/_ModalHeader.cshtml", new ModalHeaderViewModel(L("CreateNewPerson")))

<div class="modal-body">
      <form>
        <div class="my-3">
            <label for="name" class="form-label">@L("Name")</label>
            <input id="name" class="form-control" type="text" name="Name" required  maxlength="@PersonConsts.MaxNameLength">
        </div>
        <div class="my-3">
            <label for="surname" class="form-label">@L("Surname")</label>
            <input id="surname" class="form-control" type="text" name="Surname" required  maxlength="@PersonConsts.MaxSurnameLength">
        </div>
        
        <div class="my-3">
            <label for="emailAddress" class="form-label">@L("EmailAddress")</label>
            <input id="emailAddress" class="form-control" type="text" name="EmailAddress" required  maxlength="@PersonConsts.MaxEmailAddressLength">
        </div>
    </form>
</div>

@Html.Partial("~/Areas/App/Views/Common/Modals/_ModalFooterWithSaveAndCancel.cshtml")
```

Modal header and footer code comes from the template. We just changed
inside of modal-body.

We have a form with three input (name, surname and email address). We're
using **Person** entity's const values in view to set same **maxlength**
to input controls.

Also, copied the \_**Empty.js** from
**view-resources/Areas/App/Views/Common/Modals/Empty** to
**view-resources/Areas/App/Views/PhoneBook** and modified as shown below
(\_CreatePersonModal.js):

```javascript
(function($) {
    app.modals.CreatePersonModal = function () {

        var _modalManager;

        this.init = function(modalManager) {
            _modalManager = modalManager;

            //Initialize your modal here...
        };

        this.save = function () {
            //Save your modal here...
        };
    };
})(jQuery);
```

Just named the modal as '**CreatePersonModal**'. We will fill the JavaScript code later.

## Next

- [Opening the Person Modal](Developing-Step-By-Step-Core-Creating-Opening-Person-Modal.md)