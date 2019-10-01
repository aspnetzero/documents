# Filtering people

Now, we will implement **search** functionality of **GetPeople** method.
UI is shown below:

<img src="images/search-people1.png" alt="Searching people" class="img-thumbnail" />

We added a search input to **phonebook.component.html** view (showing
the related part of the code):

```html
<h3>{{"AllPeople" | localize}} ({{people.length}})</h3>
<form autocomplete="off">
    <div class="kt-form">
        <div class="row align-items-center kt-margin-b-10">
            <div class="col-xl-12">
                <div class="form-group align-items-center">
                    <div class="input-group">
                        <input [(ngModel)]="filter" name="filterText" autoFocus class="form-control" [placeholder]="l('SearchWithThreeDot' | localize)" type="text">
                        <span class="input-group-btn">
                            <button (click)="getPeople()" class="btn btn-primary" type="submit"><i class="flaticon-search-1"></i></button>
                        </span>
                    </div>
                </div>
            </div>
        </div>
    </div>
</form>


    <div  *ngFor="let person of people">
        <div class="row kt-row--no-padding align-items-center">
            <div class="col">
                <h4>{{person.name + ' ' + person.surname}}</h4>
                <span >{{person.emailAddress}}</span>
            </div>
            <div class="col kt-align-right">
                <button id="deletePerson" (click)="deletePerson(person)" title="{{'Delete' | localize}}" class="btn  btn-outline-hover-danger btn-icon" href="javascript:;">
                    <i class="fa fa-times"></i>
                </button>
            </div>
        </div>
    </div>

```

We also added currently filtered people count (people.length) in the
header. Since we have already defined and used the filter property in
**phonebook.component.ts** and implemented in the server side, this new
code immediately works.

## Next

- [Adding Phone Numbers](Developing-Step-By-Step-Angular-Adding-Phone-Numbers)