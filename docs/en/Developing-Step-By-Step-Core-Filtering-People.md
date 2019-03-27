# Filtering People

Now, we will implement **search** functionality of **GetPeople** method. UI is shown below:

<img src="images/search-people2.png" alt="Searching people" class="img-thumbnail" width="770" height="328" />

We added a search input to filter people (showing the related part of
the code):

```html
<div class="portlet light">
    <div class="portlet-title portlet-title-filter">
        <h5 class="m-subheader__title m-subheader">@L("AllPeople") (@Model.Items.Count)</h5>
        <div class="inputs inputs-full-width">
            <div class="portlet-input">
                <form action="@Url.Action("Index")" method="GET">
                    <div class="input-group">
                        <input id="FilterPeopleText" name="Filter" value="@Model.Filter" class="form-control" placeholder="@L("SearchWithThreeDot")" type="text">
                        <span class="input-group-btn">
                            <button id="FilterPeopleButton" class="btn default btn-success" type="submit"><i class="la la-search-plus"></i></button>
                        </span>
                    </div>
                </form>
            </div>
        </div>
    </div>

    <div class="portlet-body">

        ...

    </div>
</div>
```

And added Filter property to the IndexViewModel:

```csharp
public class IndexViewModel : ListResultDto<PersonListDto>
{
    public string Filter { get; set; }

    public IndexViewModel(ListResultDto<PersonListDto> output, string filter = null)
    {
        output.MapTo(this);
        Filter = filter;
    }
}
```

Lastly, changed PhoneBookController's **Index** action to pass the
**filter** to the IndexViewModel:

```csharp
public ActionResult Index(GetPeopleInput input)
{
    var output = _personAppService.GetPeople(input);
    var model = new IndexViewModel(output, input.Filter);

    return View(model);
}
```

That's all, It works! (Notice that; PersonAppService.GetPeople method
was already using the input.Filter as we implemented it before).

## Next

- [Adding Phone Numbers](Developing-Step-By-Step-Core-Adding-Phone-Numbers.md)