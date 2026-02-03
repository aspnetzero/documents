# Edit Mode For People

Now we want to edit name, surname and e-mail of people:

<img src="images/edit-person-core1.png" alt="Edit Person" class="img-thumbnail" />  

First of all, we create the necessary DTOs to transfer people's id, name,
surname and e-mail. Then we create the functions in PersonAppService for
editing people:  

```csharp
public class GetPersonForEditOutput : EntityDto<int>
{
    public string Name { get; set; }
    public string Surname { get; set; }
    public string EmailAddress { get; set; }
}

public class EditPersonInput : EntityDto<int>
{
    public string Name { get; set; }
    public string Surname { get; set; }
    public string EmailAddress { get; set; }
}
```

Then we create mappers for the edit operations. Create `PersonToGetPersonForEditOutputMapper.cs`:

```csharp
using Riok.Mapperly.Abstractions;

namespace Acme.PhoneBookDemo.PhoneBook.Mapper;

[Mapper]
public partial class PersonToGetPersonForEditOutputMapper
{
    public partial GetPersonForEditOutput Map(Person person);
}
```

Create `EditPersonInputToPersonMapper.cs` for updating existing persons:

```csharp
using Riok.Mapperly.Abstractions;

namespace Acme.PhoneBookDemo.PhoneBook.Mapper;

[Mapper]
public partial class EditPersonInputToPersonMapper
{
    public partial void Map(EditPersonInput input, Person person);
}
```

Then use ObjectMapper in the PersonAppService methods:

```csharp
public async Task<GetPersonForEditOutput> GetPersonForEdit(EntityDto input)
{
    var person = await _personRepository.GetAsync(input.Id);
    return ObjectMapper.Map<GetPersonForEditOutput>(person);
}

public async Task EditPerson(EditPersonInput input)
{
    var person = await _personRepository.GetAsync(input.Id);
    ObjectMapper.Map(input, person);
}
```

## View

Create edit-person-modal.component.html:

```html
<div
    bsModal
    #modal="bs-modal"
    (onShown)="onShown()"
    class="modal fade"
    tabindex="-1"
    role="dialog"
    aria-labelledby="modal"
    aria-hidden="true"
    [config]="{ backdrop: 'static' }">
    <div class="modal-dialog modal-lg">
        <div class="modal-content">
            @if(active()){
                <form #editPersonForm="ngForm" novalidate (ngSubmit)="save()">
                    <div class="modal-header">
                        <h4 class="modal-title">{{ 'EditPerson' | localize }}</h4>
                        <button type="button" class="btn-close" aria-label="Close" (click)="close()"></button>
                    </div>

                    <div class="modal-body">
                        <div class="mb-3">
                            <label for="name" class="form-label">{{ 'Name' | localize }}</label>
                            <input
                                #nameInput
                                id="name"
                                type="text"
                                class="form-control"
                                name="name"
                                [(ngModel)]="person().name"
                                required
                                maxlength="32" />
                        </div>

                        <div class="mb-3">
                            <label for="surname" class="form-label">{{ 'Surname' | localize }}</label>
                            <input
                                id="surname"
                                type="text"
                                class="form-control"
                                name="surname"
                                [(ngModel)]="person().surname"
                                required
                                maxlength="32" />
                        </div>

                        <div class="mb-3">
                            <label for="emailAddress" class="form-label">{{ 'EmailAddress' | localize }}</label>
                            <input
                                id="emailAddress"
                                type="email"
                                class="form-control"
                                name="emailAddress"
                                [(ngModel)]="person().emailAddress"
                                required
                                maxlength="255"
                                pattern="^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{1,})+$" />
                        </div>
                    </div>

                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" (click)="close()" [disabled]="saving()">
                            {{ 'Cancel' | localize }}
                        </button>
                        <button
                            type="submit"
                            class="btn btn-primary"
                            [disabled]="!editPersonForm.form.valid || saving()"
                            [buttonBusy]="saving()"
                            [busyText]="l('SavingWithThreeDot')">
                            <i class="fa fa-save"></i>
                            {{ 'Save' | localize }}
                        </button>
                    </div>
                </form>
            }
        </div>
    </div>
</div>
```

Add those lines to **phonebook.component.html:**:

```html 
        
	// Other Code lines...	

<button
    class="btn btn-sm btn-icon btn-bg-light btn-active-color-warning me-2"
    (click)="editPerson(person.id)"
    title="{{ 'Edit' | localize }}">
    <i class="fa fa-edit"></i>
</button>
                
	// Other Code lines...
				
<createPersonModal #createPersonModal(modalSave)="getPeople()"></createPersonModal>
<editPersonModal #editPersonModal (modalSave)="getPeople()"></editPersonModal>
```

Add those lines to **phonebook.component.ts:**:

```typescript 
@Component({
    selector: 'app-phone-book',
    templateUrl: './phonebook.component.html',
    animations: [appModuleAnimation()],
    imports: [
      //..
      EditPersonModalComponent
    ],
})
export class PhoneBookComponent extends AppComponentBase implements OnInit {
    //..
    editPersonModal = viewChild.required<EditPersonModalComponent>('editPersonModal');
    
    //..
    editPerson(personId: number): void {
        this.editPersonModal().show(personId);
    }
}
```


## Controller

Create edit-person-modal.component.ts:

```typescript
import { Component, ElementRef, inject, signal, viewChild, output } from '@angular/core';
import { ModalDirective, ModalModule } from 'ngx-bootstrap/modal';
import { PersonServiceProxy, GetPersonForEditOutput, EditPersonInput } from '@shared/service-proxies/service-proxies';
import { AppComponentBase } from '@shared/common/app-component-base';
import { finalize } from 'rxjs/operators';
import { FormsModule } from '@angular/forms';
import { LocalizePipe } from '@shared/common/pipes/localize.pipe';
import { ButtonBusyDirective } from '@shared/utils/button-busy.directive';
import { CommonModule } from '@angular/common';

@Component({
    selector: 'editPersonModal',
    templateUrl: './edit-person-modal.component.html',
    standalone: true,
    imports: [CommonModule, FormsModule, ModalModule, LocalizePipe, ButtonBusyDirective],
})
export class EditPersonModalComponent extends AppComponentBase {
    private readonly _personService = inject(PersonServiceProxy);

    modalSave = output<any>();

    modal = viewChild.required<ModalDirective>('modal');
    nameInput = viewChild<ElementRef>('nameInput');

    person = signal(new GetPersonForEditOutput());
    active = signal(false);
    saving = signal(false);

    constructor() {
        super();
    }

    show(id: number): void {
        this._personService.getPersonForEdit(id).subscribe((result) => {
            this.person.set(result);
            this.active.set(true);
            this.modal().show();
        });
    }

    onShown(): void {
        this.nameInput()?.nativeElement.focus();
    }

    save(): void {
        this.saving.set(true);

        const input = new EditPersonInput();
        const currentPerson = this.person();
        input.id = currentPerson.id;
        input.name = currentPerson.name;
        input.surname = currentPerson.surname;
        input.emailAddress = currentPerson.emailAddress;

        this._personService
            .editPerson(input)
            .pipe(finalize(() => this.saving.set(false)))
            .subscribe(() => {
                this.notify.success(this.l('SavedSuccessfully'));
                this.close();
                this.modalSave.emit(null);
            });
    }

    close(): void {
        this.modal().hide();
        this.active.set(false);
    }
}
```

## Next

- [Multi Tenancy](Developing-Step-By-Step-Angular-Multi-Tenancy)