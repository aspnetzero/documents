# Filtering People

Now, we will implement **search** functionality of **GetPeople** method. UI is shown below:

<img src="images/search-people-3.png" alt="Searching people" class="img-thumbnail" width="770" height="328" />

We added a search input to filter people (showing the related part of
the code):

```html
<div class="card-body">
	<div class="col-xl-12">
		<div class="mb-5 align-items-center">
			<form action="@Url.Action("Index")" method="GET">
				<div class="input-group">
					<input type="text" id="UsersTableFilter" name="Filter" value="@Model.Filter"  class="form-control" placeholder="@L("SearchWithThreeDot")">
					<button id="GetUsersButton" class="btn btn-primary" type="submit">
						<i class="flaticon-search-1" aria-label="Search"></i>
					</button>
				</div>
			</form>
		</div>
	</div>
<!--...--->
```

And added Filter property to the IndexViewModel:

```csharp
public class IndexViewModel : ListResultDto<PersonListDto>
{
    public string Filter { get; set; }
}
```

Lastly, changed PhoneBookController's **Index** action to pass the
**filter** to the IndexViewModel:

```csharp
public ActionResult Index(GetPeopleInput input)
{
    var output = _personAppService.GetPeople(input);
    var model = ObjectMapper.MapTo<IndexViewModel>(output);
    model.Filter = input.Filter;

    return View(model);
}
```

That's all, It works! (Notice that; PersonAppService.GetPeople method
was already using the input.Filter as we implemented it before).

## Next

- [Adding Phone Numbers](Developing-Step-By-Step-Core-Adding-Phone-Numbers.md)