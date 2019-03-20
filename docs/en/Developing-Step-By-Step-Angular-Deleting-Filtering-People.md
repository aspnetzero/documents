# Filtering people

Now, we will implement **search** functionality of **GetPeople** method.
UI is shown below:

<img src="D:/Github/documents/docs/en/images/search-people1.png" alt="Searching people" class="img-thumbnail" />

We added a search input to **phonebook.component.html** view (showing
the related part of the code):

```html
<h3>{{l("AllPeople" | localize)}} ({{people.length}})</h3>
<form autocomplete="off">
    <div class="m-form m-form--label-align-right">
        <div class="row align-items-center m--margin-bottom-10">
            <div class="col-xl-12">
                <div class="form-group m-form__group align-items-center">
                    <div class="input-group">
                        <input [(ngModel)]="filter" name="filterText" autoFocus class="form-control m-input" [placeholder]="l('SearchWithThreeDot' | localize)" type="text">
                        <span class="input-group-btn">
                            <button (click)="getPeople()" class="btn btn-primary" type="submit"><i class="flaticon-search-1"></i></button>
                        </span>
                    </div>
                </div>
            </div>
        </div>
    </div>
</form>

<div class="m-widget1">
    <div class="m-widget1__item" *ngFor="let person of people">
        <div class="row m-row--no-padding align-items-center">
            <div class="col">
                <h3 class="m-widget1__title">{{person.name + ' ' + person.surname}}</h3>
                <span class="m-widget1__desc">{{person.emailAddress}}</span>
            </div>
            <div class="col m--align-right">
                <button id="deletePerson" (click)="deletePerson(person)" title="{{l('Delete' | localize)}}" class="btn btn-outline-danger m-btn m-btn--icon m-btn--icon-only m-btn--pill" href="javascript:;">
                    <i class="fa fa-times"></i>
                </button>
            </div>
        </div>
    </div>
</div>
```

We also added currently filtered people count (people.length) in the
header. Since we have already defined and used the filter property in
**phonebook.component.ts** and implemented in the server side, this new
code immediately works.