# Edit Mode For People

Now we want to edit name, surname and e-mail of people:

<img src="images/edit-person-core-2.png" alt="Edit Person" class="img-thumbnail" />  

First of all, we create the necessary DTOs to transfer people's id, name,
surname and e-mail. We can optionally configure auto-mapper, but this is not necessary because all properties match automatically. Then we create the functions in PersonAppService for
editing people:  

```csharp
[AbpAuthorize(AppPermissions.Pages_Tenant_PhoneBook_EditPerson)]
public async Task EditPerson(EditPersonInput input)
{
    var person = await _personRepository.GetAsync(input.Id);
    person.Name = input.Name;
    person.Surname = input.Surname;
    person.EmailAddress = input.EmailAddress;
    await _personRepository.UpdateAsync(person);
}
```

Add following tab part to **EditPersonModal.html**

```html
<tab class="pt-5" heading="{{ 'Person' | localize }}">
    <div class="my-3">
        <label class="form-label">{{"Name" | localize}}</label>
        <input #nameInput class="form-control" type="text" name="name" [(ngModel)]="personToEdit.name"
                required maxlength="32">
    </div>
    <div class="my-3">
        <label class="form-label">{{"Surname" | localize}}</label>
        <input class="form-control" type="email" name="surname" [(ngModel)]="personToEdit.surname" required
                maxlength="32">
    </div>
    <div class="my-3">
        <label class="form-label">{{"EmailAddress" | localize}}</label>
        <input class="form-control" type="email" name="emailAddress" [(ngModel)]="personToEdit.emailAddress"
                required maxlength="255" pattern="^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{1,})+$">
    </div>
    <div class="my-3">
        <button class="btn btn-primary" type="button" (click)="savePerson()">{{'Save' | localize}}</button>
    </div>
</tab>
```

And create **savePerson** method in to **EditPersonModal.ts**

Add those lines to **phonebook.component.html:**:

```typescript
savePerson(): void {
    this.saving = true;
    this._personService.editPerson(new EditPersonInput({
        id: this.personToEdit.id,
        name: this.personToEdit.name,
        surname: this.personToEdit.surname,
        emailAddress: this.personToEdit.emailAddress
    }))
        .pipe(finalize(() => this.saving = false))
        .subscribe(() => {
            this.notify.info(this.l('SavedSuccessfully'));
            this.hasChanges = true;
        });
}
```

When you click **Edit** button in index, now your person edit tab will also work.

## Next

- [Multi Tenancy](Developing-Step-By-Step-Angular-Multi-Tenancy)