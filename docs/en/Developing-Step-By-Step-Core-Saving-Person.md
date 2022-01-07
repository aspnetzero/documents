# Saving Person

Finally, we can save the person when user clicks the the save button. We
implement it in the \_CreatePersonModal.js file:

```javascript
(function($) {
    app.modals.CreatePersonModal = function () {
        var _modalManager;
        var _personService = abp.services.app.person;
        var _$form = null;

        this.init = function(modalManager) {
            _modalManager = modalManager;

            _$form = _modalManager.getModal().find('form');
            _$form.validate();
        };

        this.save = function () {
            if (!_$form.valid()) {
                return;
            }

            var person = _$form.serializeFormToObject();

            _modalManager.setBusy(true);
            
            _personService.createPerson(person)
                .done(function () {
                    _modalManager.close();
                    abp.notify.info(app.localize('SavedSuccessfully'));

                    abp.event.trigger('app.createPersonModalSaved');//trigger an event here to let your app know that your create person is completed.
                })
                .always(function () {
                    _modalManager.setBusy(false);
                });
        };
    };
})(jQuery);

```

In the **init** function, we got a reference to the **form** and enable
**validation**.

In the **save** method, we first **validate** the **form**. Then we
convert the form to a **javascript object** (since server waits for
JSON). Then we use PersonAppService's **createPerson** method to save
the person. **setBusy** method automatically disables save and cancel
buttons. In the **done** method, we simple **reload**ed the page to show
new user in the page (In a real application, we could add new person to
the page dynamically using jQuery)

Notice that; We used PersonAppService's createPerson method directly
from javascript. This is possible by ABP's [dynamic javascript proxy
system](https://aspnetboilerplate.com/Pages/Documents/AspNet-Core#client-proxies).

Now go to **Index.js** and refresh data when **app.createPersonModalSaved** triggered.

_Index.js_
```javascript
        /...
        function getPeople(){
            dataTable.ajax.reload();
        }
        
        abp.event.on('app.createPersonModalSaved', getPeople); 
    });
})();
```

## Next

- [Authorization For Phone Book](Developing-Step-By-Step-Core-Authorization-For-Phone-Book.md)