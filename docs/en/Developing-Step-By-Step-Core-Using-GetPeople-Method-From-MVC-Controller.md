# Using GetPeople Method From MVC Controller

It's time to open **PhoneBookController** and get people to show on the
view:

```csharp
[Area("App")]
public class PhoneBookController : PhoneBookDemoControllerBase
{
    private readonly IPersonAppService _personAppService;

    public PhoneBookController(IPersonAppService personAppService)
    {
        _personAppService = personAppService;
    }

    public ActionResult Index(GetPeopleInput input)
    {
        var output = _personAppService.GetPeople(input);
        var model = ObjectMapper.MapTo<IndexViewModel>(output);

        return View(model);
    }
}
```

We inject **IPersonAppService** and call its **GetPeople** method
(which is created and tested before) to get list of people. Then we
created a ViewModel object and passes to the view. Let's see the
**IndexViewModel** class:

```csharp
[AutoMapFrom(typeof(ListResultDto<PersonListDto>))]
public class IndexViewModel : ListResultDto<PersonListDto>
{

}
```

Here, we're extending the output of the PersonAppService.GetPeople method.
We get the output from the constructor and map it to this object.

## Application Services and ViewModels

We created an **Application Service** (PersonAppService) and used it
from the **Controller**. Instead, we could access **Repository**
directly from Controller and completely discard the application service.
ASP.NET Zero does not enforce any architecture here. In ASP.NET Zero, we
use the application layer (application services and DTOs). Therefore, we
implemented it **independent** from ASP.NET MVC. This makes application
layer re-usable from different presentation layers. But if you will only
develop ASP.NET MVC, you can implement application logic inside
controllers and access to the repositories from controllers. This may
simplify your architecture and development model.

If you decide to develop application services and use them in
controllers then you can use application service's **output** as your
view model. We did not prefer it and wrapped output by a dedicated
**ViewModel** (IndexViewModel here) since we think that we may add some
**additional properties/methods** for our view model. Again, it's your
choice of implementation.

# Rendering People In MVC View

We show people on the page is most basic form. See the changed view
below:

```html
...
@model Acme.PhoneBookDemo.Web.Areas.App.Models.PhoneBook.IndexViewModel
...
        <h5 class="kt-subheader__title">@L("AllPeople")</h5>

    <div class="list-group">
    @foreach (var person in Model.Items)
    {
        <a href="javascript:;" style="color: black" class="list-group-item">
            <h5 class="list-group-item-heading" style="font-weight: lighter">
                @person.Name @person.Surname
            </h5>
            <p class="list-group-item-text">
                @person.EmailAddress
            </p>
        </a>
    }
    </div>
...
```

We declared the **@model** and used a foreach loop to render people. See
the result:

<img src="images/phonebook-people-view-3.png" alt="Phonebook peoples" class="img-thumbnail" width="954" height="354" />

We successfully retrieved list of people from database to the page.

## About Showing Tabular Data

We normally use a javascript based rich table/grid library to show
tabular data, instead of manually rendering data like that. For example,
we used to use [datatables](https://datatables.net/) library to show users on the
Users page of ASP.NET Zero. Always use such components since they make
things much more easier and provides a much better user experience.

We did not use a table component here, because we want to show basics of
MVC instead of going details of a 3rd party library.

## Next

- [Creating a New Person](Developing-Step-By-Step-Core-Creating-New-Person.md)
