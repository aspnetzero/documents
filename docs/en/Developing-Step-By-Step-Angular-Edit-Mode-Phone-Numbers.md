# Edit Mode For Phone Numbers

We will create an edit modal which has two tabs for person (first tab for **Person** second tab for **Phones for that person**).Lets implement editing phone numbers first. Final UI is shown below:

<img src="images/phone-book-edit-mode1.png" alt="Phone book edit mode" class="img-thumbnail" />

# Creating EditPersonModal

## View

Create new component named **edit-person-modal.component.html** in phonebook folder and change it's view as seen below:

```html
<div bsModal #modal="bs-modal" class="modal fade" tabindex="-1" role="dialog"
     aria-labelledby="modal" aria-hidden="true" [config]="{backdrop: 'static'}">
    <div class="modal-dialog modal-lg">
        <div class="modal-content">
            <div class="modal-header">
                <h4 class="modal-title">
                    <span>{{"EditPerson" | localize}}</span>
                </h4>
                <button
                    type="button"
                    class="btn-close"
                    (click)="close()"
                    [attr.aria-label]="l('Close')"
                    [disabled]="saving"
                >
                </button>
            </div>
            <div class="modal-body">
                <tabset *ngIf="personToEdit">
                    <tab class="pt-5" heading="{{ 'Person' | localize }}">
                        Person Edit Will Be Here
                    </tab>
                    <tab class="pt-5" heading="{{ 'Phones' | localize }}">
                        <form>
                            <div class="input-group mb-3">
                                <div class="input-group-prepend">
                                    <select name="Type" class="form-control" [(ngModel)]="addPhoneInput.type">
                                        <option value="0" selected>{{'PhoneType_Mobile' | localize}}</option>
                                        <option value="1">{{'PhoneType_Home' | localize}}</option>
                                        <option value="2">{{'PhoneType_Business' | localize}}</option>
                                    </select>
                                </div>
                                <input class="form-control" type="text" name="Number" [(ngModel)]="addPhoneInput.number"
                                       [placeholder]="l('PhoneNumber')" required>
                                <div class="input-group-append">
                                    <button class="btn btn-success" (click)="addPhone()" type="button">
                                        {{'AddPhone' | localize}}
                                    </button>
                                </div>
                            </div>
                        </form>
                        <div class="row">
                            <table class="table align-middle table-row-dashed fs-6 gy-5 dataTable no-footer">
                                <tbody>
                                    <tr *ngFor="let phone of personToEdit.phones">
                                        <td>{{getPhoneTypeString(phone.type)}}</td>
                                        <td>{{phone.number}}</td>
                                        <td>
                                            <button class="btn btn-danger btn-sm" (click)="deletePhone(phone.id)">
                                                <i class="la la-floppy-o"></i>
                                                {{'Delete' | localize}}
                                            </button>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </tab>
                </tabset>
            </div>
            <div class="modal-footer">
                <button
                    [disabled]="saving"
                    type="button"
                    class="btn btn-light-primary font-weight-bold"
                    (click)="close()"
                >
                    {{ 'Close' | localize }}
                </button>
            </div>
        </div>
    </div>
</div>

```

## Edit Person Component Class

Create new component named **edit-person-modal.component.ts** in phonebook folder and change it's view as seen below:

```typescript
import {Component, ViewChild, Injector, ElementRef, Output, EventEmitter} from '@angular/core';
import {ModalDirective} from 'ngx-bootstrap/modal';
import {PersonServiceProxy, AddPhoneInput, PersonListDto, PhoneType} from '@shared/service-proxies/service-proxies';
import {AppComponentBase} from '@shared/common/app-component-base';
import {finalize} from 'rxjs/operators';

@Component({
    selector: 'editPersonModal',
    templateUrl: './edit-person-modal.component.html',
})
export class EditPersonModalComponent extends AppComponentBase {
    @Output() onClosedWithChanges: EventEmitter<any> = new EventEmitter<any>();

    @ViewChild('modal', {static: false}) modal: ModalDirective;

    hasChanges: boolean = false;
    active: boolean = false;
    saving: boolean = false;

    addPhoneInput: AddPhoneInput = new AddPhoneInput();
    personToEdit: PersonListDto;

    constructor(
        injector: Injector,
        private _personService: PersonServiceProxy
    ) {
        super(injector);
    }

    show(personId: number): void {
        this.active = true;
        this.addPhoneInput = new AddPhoneInput();
        this._personService.getPerson(personId).subscribe((person) => {
            this.personToEdit = person;
            this.modal.show();
        });
    }

    close(): void {
        this.modal.hide();
        this.active = false;
        if (this.hasChanges) {
            this.onClosedWithChanges.emit(null);
        }
    }

    addPhone() {
        this.saving = true;
        this.addPhoneInput.personId = this.personToEdit.id;

        this._personService.addPhone(this.addPhoneInput)
            .pipe(
                finalize(() => {
                    this.saving = false;
                })
            )
            .subscribe((callback) => {
                abp.notify.info(this.l('SavedSuccessfully'));
                this.hasChanges = true;
                this.personToEdit.phones.push(callback);
            });
    }

    deletePhone(phoneId) {
        if (!phoneId) {
            return;
        }

        this.message.confirm(
            this.l('DeletePhoneWarningMessage'),
            this.l('AreYouSure'),
            isConfirmed => {
                if (isConfirmed) {
                    this.saving = true;
                    this._personService.deletePhone(phoneId)
                        .pipe(
                            finalize(() => {
                                this.saving = false;
                            })
                        )
                        .subscribe(() => {
                            abp.notify.info(this.l('SavedSuccessfully'));
                            this.hasChanges = true;
                            this.personToEdit.phones = this.personToEdit.phones.filter(p => p.id != phoneId);
                        })
                }
            }
        );
    }


    getPhoneTypeString(phoneType: PhoneType): string {
        switch (phoneType) {
            case 1:
                return this.l('Home');
            case 2:
                return this.l('Business');
            default:
                return this.l('Mobile');
        }
    }
}

```

## Update PhoneBookModule

We should add **EditPersonModalComponent** to **PhoneBookModule**.

```typescript
import { NgModule } from '@angular/core';
import { AppSharedModule } from '@app/shared/app-shared.module';
import { PhoneBookRoutingModule } from './phone-book-routing.module';
import { PhoneBookComponent } from './phone-book.component';
import { SubheaderModule } from '@app/shared/common/sub-header/subheader.module';
import { CreatePersonModalComponent } from './create-person-modal.component';
import { EditPersonModalComponent } from './edit-person-modal.component';

@NgModule({
    declarations: [PhoneBookComponent, CreatePersonModalComponent, EditPersonModalComponent],
    imports: [AppSharedModule, PhoneBookRoutingModule, SubheaderModule],
})
export class PhoneBookModule {}
```

## View

Now we can add edit button to index view and call edit person modal on button click. Changes in view are shown below:

```html
<!--...-->
<ul
    id="dropdownMenu"
    class="dropdown-menu"
    role="menu"
    *dropdownMenu
    aria-labelledby="dropdownButton"
>
    <li
        *ngIf="'Pages.Tenant.PhoneBook.EditPerson' | permission"
        role="menuitem"
    >
        <a
            href="javascript:;"
            class="dropdown-item"
            (click)="editPersonModal.show(record.id)"
        >
            {{ 'Edit' | localize }}
        </a>
    </li>

<!--...-->
    <createPersonModal #createPersonModal (modalSave)="getPeople()"></createPersonModal>
    <editPersonModal #editPersonModal (onClosedWithChanges)="getPeople()"></editPersonModal><!--Add edit-->
```

## Next

- [Edit Mode For People](Developing-Step-By-Step-Angular-Edit-Mode-People)
