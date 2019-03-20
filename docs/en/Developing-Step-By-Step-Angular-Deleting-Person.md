# Deleting a Person

Let's add a delete button in people list as shown below:

<img src="D:/Github/documents/docs/en/images/phonebook-people-delete-button1.png" alt="Delete person" class="img-thumbnail" />

We're starting from UI in this case.

## View

We're changing **phonebook.component.html** view to add a delete button
(related part is shown here):

```html
...
<h3>{{l("AllPeople" | localize)}}</h3>
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
...
```

We simply added a button which calls **deletePerson** method (will be
defined) when it's clicked. You can define a permission for 'deleting
person' as we did for 'creating person' above.

## Style

We're using **[LESS](http://lesscss.org/)** files for styling the components. We created a file named **phonebook.component.less** (in
phonebook folder) with an empty content.

```css
/* styles */
```

And adding the style to the **phonebook.component.ts** Component
declaration:

```typescript
@Component({
    templateUrl: './phonebook.component.html',
    styleUrls: ['./phonebook.component.less'],
    animations: [appModuleAnimation()]
})
```

Now, we can now see the buttons, but they don't work since we haven't
defined the deletePerson method yet.

## Application Service

Let's leave the client side and add a DeletePerson method to the server
side. We are adding it to the service interface,**IPersonAppService:**:

```csharp
Task DeletePerson(EntityDto input);
```

**EntityDto** is a shortcut of ABP if we only get an id value.
Implementation (in **PersonAppService**) is very simple:

```csharp
[AbpAuthorize(AppPermissions.Pages_Tenant_PhoneBook_DeletePerson)]
public async Task DeletePerson(EntityDto input)
{
    await _personRepository.DeleteAsync(input.Id);
}
```

## Service Proxy Generation

Since we changed server side services, we should re-generate the client
side service proxies via NSwag. Make server side running and use
refresh.bat as we did before.

## Component Script

Now, we can add **deletePerson** method to **phonebook.component.ts**:

```typescript
deletePerson(person: PersonListDto): void {
    this.message.confirm(
        this.l('AreYouSureToDeleteThePerson' | localize: person.name),
        isConfirmed => {
            if (isConfirmed) {
                this._personService.deletePerson(person.id).subscribe(() => {
                    this.notify.info(this.l('SuccessfullyDeleted' | localize));
                    _.remove(this.people, person);
                });
            }
        }
    );
} 
```

It first shows a confirmation message when we click the delete button:

<img src="D:/Github/documents/docs/en/images/confirmation-delete-person1.png" alt="Confirmation message" class="img-thumbnail" />

If we click Yes, it simply calls **deletePerson** method of
**PersonAppService** and shows a
**[notification](https://aspnetboilerplate.com/Pages/Documents/Javascript-API/Notification)**
if operation succeed. Also, removes the person from the person array
using [lodash](https://lodash.com/) library. We also added an import
statement before the @Component declaration:

```typescript
import * as _ from 'lodash';
```

