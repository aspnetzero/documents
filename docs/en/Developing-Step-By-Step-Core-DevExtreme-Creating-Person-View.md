# Creating New Person

Create a controller action to create a person.

(PhoneBookController.cs): 

```cs
[Area("App")]
public class PhoneBookController : PhoneBookDemoControllerBase
{
    private readonly IPersonAppService _personAppService;

    public PhoneBookController(IPersonAppService personAppService)
    {
        _personAppService = personAppService;
    }
    
    //...
    public async Task<IActionResult> CreatePerson(string values)
    {
        var input = new CreatePersonInput();
        JsonConvert.PopulateObject(values, input);

        if(!TryValidateModel(input))
            return BadRequest(ModelState.GetFullErrorMessage());

        await _personAppService.CreatePerson(input);
        return Ok();
    }
}
```

Then Update **Index.cshml** as seen below:

(Index.cshtml)

```html
@using Acme.PhoneBookDemo.Web.Areas.App.Startup
@using DevExtreme.AspNet.Mvc
@{
    ViewBag.CurrentPageName = AppPageNames.Tenant.PhoneBook;
}
<div class="content d-flex flex-column flex-column-fluid" id="kt_content">
    <abp-page-subheader title="@L("PhoneBook")" description="@L("PhoneBookInfo")"></abp-page-subheader>

    <div class="@(await GetContainerClass())">
        <div class="col-12">
            <div class="card card-custom gutter-b">
                <div class="card-body">
                    @(Html.DevExtreme()
                        .DataGrid()              
                        .Editing(editing => {
                            editing.Mode(GridEditMode.Row);
                            editing.AllowAdding(true);
                        })
                        .DataSource(d => d.Mvc()
                            .Controller("PhoneBook")
                            .LoadAction("LoadPeople")
                            .InsertAction("CreatePerson")
                            .Key("id")
                        )
                        .Columns(columns =>
                        {
                            columns.Add()
                                .DataField("name")
                                .Caption(L("Name"));
                            
                            columns.Add()
                                .DataField("surname")
                                .Caption(L("Surname"));
                            
                            columns.Add()
                                .DataField("emailAddress")
                                .Caption(L("EmailAddress"));                       
                        })
                    )
                </div>
            </div>
        </div>
    </div>
</div>
```

<img src="images/phonebook-devextreme-create-person.png" alt="Phonebook peoples" class="img-thumbnail"/>

## Next

- [Deleting a Person](Developing-Step-By-Step-Core-DevExtreme-Deleting-Person.md)

