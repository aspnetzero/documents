# Edit Mode For People

Now we want to edit name, surname and e-mail of people:

<img src="images/edit-person-core-2.png" alt="Edit Person" class="img-thumbnail" />  

## Edit Person Method

We are adding **EditPerson** method to IPersonAppService interface as shown
below:

```csharp
Task EditPerson(EditPersonInput input);
```

EditPhoneInput DTO is shown below:

```csharp
public class EditPersonInput
{
    [Required]
    public int Id { get; set; }
    
    [Required]
    [MaxLength(PersonConsts.MaxNameLength)]
    public string Name { get; set; }
    
    [Required]
    [MaxLength(PersonConsts.MaxSurnameLength)]
    public string Surname { get; set; }
    
    [Required]
    [MaxLength(PersonConsts.MaxEmailAddressLength)]
    public string EmailAddress { get; set; }
}
```

Now, we can implement this method as shown below:

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

Then run **nswag/refresh.bat** file on the client side to re-generate service proxy classes. 

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

Add those lines to **phonebook.component.html:**

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