# Edit Mode For People

First of all, we create the necessary DTOs to transfer people's id, name, surname and e-mail.

```cs
public class EditPersonInput : EntityDto
{
    public string Name { get; set; }

    public string Surname { get; set; }

    public string EmailAddress { get; set; }
}
```

Then create the functions in **PersonAppService** for editing people:  

```csharp
public async Task EditPerson(EditPersonInput input)
{
    var person = await _personRepository.GetAsync(input.Id);
    if (input.Name != null)
    {
        person.Name = input.Name;
    }

    if (input.Surname != null)
    {
        person.Surname = input.Surname;
    }

    if (input.EmailAddress != null)
    {
        person.EmailAddress = input.EmailAddress;
    }

    await _personRepository.UpdateAsync(person);
}
```

Then add **UpdatePerson** method in **PhoneBookController**

```cs
[HttpPut]
public async Task UpdatePerson(int key, string values)
{
    var editPersonInput = new EditPersonInput {Id = key};
    JsonConvert.PopulateObject(values, editPersonInput);

    await _personAppService.EditPerson(editPersonInput);
}
```



## View

Update **Index.cshtml** as seen below.

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
                            editing.AllowDeleting(true);
                            editing.AllowUpdating(true);
                        })
                        .DataSource(d => d.Mvc()
                            .Controller("PhoneBook")
                            .LoadAction("LoadPeople")
                            .InsertAction("CreatePerson")
                            .UpdateAction("UpdatePerson")
                            .DeleteAction("DeletePerson")
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

<img src="images/devextreme-confirmation-edit-person.png" alt="Phonebook peoples" class="img-thumbnail"/>

## Next

- [Multi Tenancy](Developing-Step-By-Step-Core-Multi-Tenancy.md)