# Creating the Page

After creating the menu item, we can create an empty page.

## Controller

Creating the **PhoneBookController** under **Areas/App/Controllers**
folder in the Web project:

```csharp
[Area("App")]
public class PhoneBookController : PhoneBookDemoControllerBase
{
    public ActionResult Index()
    {
        return View();
    }
}
```

We inherited from **PhoneBookDemoControllerBase** (will be
*YourProjectName*ControllerBase for your projects) instead of MVC's
standard Controller class. While it will work if we derive from the
standard Controller, PhoneBookDemoControllerBase provides very useful
base properties and methods. So, always inherit from this class unless
it has a disadvantage for your case.

## View

Creating an empty view, **Index.cshtml** under
**Areas/App/Views/PhoneBook** folder:

```html
@using System.Threading.Tasks
@using Acme.PhoneBookDemo.Web.Areas.App.Startup

@{
ViewBag.CurrentPageName = AppPageNames.Tenant.PhoneBook;
}

<div class="kt-content  kt-grid__item kt-grid__item--fluid kt-grid kt-grid--hor">
    <div class="kt-subheader kt-grid__item">
        <div class="kt-container ">
            <div class="kt-subheader__main">
                <h3 class="kt-subheader__title">
                    <span>@L("PhoneBook")</span>
                </h3>
                <span class="kt-subheader__separator kt-subheader__separator--v"></span>
                <span class="kt-subheader__desc">
                        @L("PhoneBookInfo")
                </span>
            </div>
        </div>
    </div>
    <div class="kt-container kt-grid__item kt-grid__item--fluid">
        <p>PHONE BOOK CONTENT COMES HERE!</p>
    </div>
</div>
```

We set ViewBag.CurrentPageName to the current page's name to
automatically highlight the related menu item when this page is active.
Now, it's time to run application and see the new phone book page:

<img src="images/phonebook-empty-mpa2.png" alt="Phone book empty screen" class="img-thumbnail" />

Menu item display name and page title are localized. Try to change UI
language to see difference.

## Next

- [Creating Person Entity](Developing-Step-By-Step-Core-DevExtreme-Creating-Person-Entity.md)
