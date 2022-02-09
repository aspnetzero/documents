# Creating Modal for New Person

We will create a Bootstrap Modal to create a new person. ASP.NET Zero
uses [ngx-bootstrap](https://github.com/valor-software/ngx-bootstrap)
library to create modals (you can use another library, but we will use
it in this sample too). Final modal will be like below:

<img src="images/phonebook-create-person-dialog1.png" alt="Create Person Dialog" class="img-thumbnail" />

First of all, we should use **nswag/refresh.bat** to re-generate
service-proxies. This will generate the code that is needed to call
PersonAppService.**CreatePerson** method from client side. Notice that
you should rebuild & run the server side application before
re-generating the proxy scripts.

We are starting from creating a new component, named
**create-person-modal.component.ts** into client side phonebook folder:

```typescript
import { Component, ViewChild, Injector, ElementRef, Output, EventEmitter } from '@angular/core';
import { ModalDirective } from 'ngx-bootstrap/modal';
import { PersonServiceProxy, CreatePersonInput } from '@shared/service-proxies/service-proxies';
import { AppComponentBase } from '@shared/common/app-component-base';
import { finalize } from 'rxjs/operators';

@Component({
    selector: 'createPersonModal',
    templateUrl: './create-person-modal.component.html'
})
export class CreatePersonModalComponent extends AppComponentBase {

    @Output() modalSave: EventEmitter<any> = new EventEmitter<any>();

    @ViewChild('modal' , { static: false }) modal: ModalDirective;
    @ViewChild('nameInput' , { static: false }) nameInput: ElementRef;

    person: CreatePersonInput = new CreatePersonInput();

    active: boolean = false;
    saving: boolean = false;

    constructor(
        injector: Injector,
        private _personService: PersonServiceProxy
    ) {
        super(injector);
    }

    show(): void {
        this.active = true;
        this.person = new CreatePersonInput();
        this.modal.show();
    }

    onShown(): void {
        this.nameInput.nativeElement.focus();
    }

    save(): void {
        this.saving = true;
        this._personService.createPerson(this.person)
            .pipe(finalize(() => this.saving = false))
            .subscribe(() => {
                this.notify.info(this.l('SavedSuccessfully'));
                this.close();
                this.modalSave.emit(this.person);
            });
    }

    close(): void {
        this.modal.hide();
        this.active = false;
    }
}

```

Let me explain some parts of this class:

- It has a selector, **createPersonModal**, which will be used as like
  an HTML element in the person list page.
- It extends **AppComponentBase** to take advantage of it (defines
  this.l and this.notify in this sample).
- Defines an event, **modalSave**, which is triggered when we
  successfully save the modal. Thus, the main page will be informed
  and it can reload the person list.
- Declares two **ViewChild** members (**modal** and **nameInput**) to
  access some elements in the view.
- Injects **PersonServiceProxy** to call server side method while
  creating the person.
- It focuses to **name** input when modal is shown.

The code is simple and easy to understand except a small hack: an active
flag is used to reset validation for Angular view (explained in
angular's
[documentation](https://angular.io/docs/ts/latest/cookbook/form-validation.html)).

As declared in the component, we are creating the
**create-person-modal.component.html** file in the same folder as shown
below:

```html
<div bsModal #modal="bs-modal" (onShown)="onShown()" class="modal fade" tabindex="-1" role="dialog"
     aria-labelledby="modal" aria-hidden="true" [config]="{backdrop: 'static'}">
    <div class="modal-dialog modal-lg">
        <div class="modal-content">
            <form *ngIf="active" #personForm="ngForm" novalidate (ngSubmit)="save()">
                <div class="modal-header">
                    <h4 class="modal-title">
                        <span>{{"CreateNewPerson" | localize}}</span>
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
                    <div class="my-3">
                        <label class="form-label">{{"Name" | localize}}</label>
                        <input #nameInput class="form-control" type="text" name="name" [(ngModel)]="person.name"
                               required maxlength="32">
                    </div>
                    <div class="my-3">
                        <label class="form-label">{{"Surname" | localize}}</label>
                        <input class="form-control" type="email" name="surname" [(ngModel)]="person.surname" required
                               maxlength="32">
                    </div>
                    <div class="my-3">
                        <label class="form-label">{{"EmailAddress" | localize}}</label>
                        <input class="form-control" type="email" name="emailAddress" [(ngModel)]="person.emailAddress"
                               required maxlength="255" pattern="^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{1,})+$">
                    </div>
                </div>
                <div class="modal-footer">
                    <button
                        [disabled]="saving"
                        type="button"
                        class="btn btn-light-primary font-weight-bold"
                        (click)="close()"
                    >
                        {{ 'Cancel' | localize }}
                    </button>
                    <button
                        type="submit"
                        class="btn btn-primary font-weight-bold"
                        [disabled]="!personForm.form.valid"
                        [buttonBusy]="saving"
                        [busyText]="l('SavingWithThreeDot')"
                    >
                        <i class="fa fa-save"></i>
                        <span>{{ 'Save' | localize }}</span>
                    </button>
                </div>
            </form>
        </div>
    </div>
</div>

```

Most of this code is similar for all modals. The important part is how
we binded model to the view using the ngModel directive. As like all
components, Angular requires to relate it to a module. We should add it to
**declarations** array of **phonebook.module.ts** as like shown below:

```typescript
import {NgModule} from '@angular/core';
import {AppSharedModule} from '@app/shared/app-shared.module';
import {PhoneBookRoutingModule} from './phonebook-routing.module';
import {PhoneBookComponent} from './phonebook.component';
import {SubheaderModule} from "@app/shared/common/sub-header/subheader.module";
import { CreatePersonModalComponent } from './create-person-modal.component';

@NgModule({
    declarations: [PhoneBookComponent, CreatePersonModalComponent],
    imports: [AppSharedModule, SubheaderModule, PhoneBookRoutingModule]
})
export class PhoneBookModule {}
```

We need to put a "Create new person" button to the 'people list page' to
open the modal when clicked to the button. To do that, we made the
following changes in **phonebook.component.html**:

```html
<div [@routerTransition]>
    <div class="content d-flex flex-column flex-column-fluid">
        <sub-header [title]="'PhoneBook' | localize" [description]="'PhoneBookInfo' | localize">
            <div role="actions"><!--Add Actions-->
                <button
                    (click)="createPersonModal.show()"
                    class="btn btn-primary"
                >
                    <i class="fa fa-plus"></i>
                    {{ 'CreateNewPerson' | localize }}
                </button>
            </div>
        </sub-header>
        <div [class]="containerClass">
            <div class="card card-custom gutter-b">
                <div class="card-body">
                    <div class="row align-items-center">
                        <!--<Primeng-TurboTable-Start>-->
                        <div class="primeng-datatable-container" [busyIf]="primengTableHelper.isLoading">
                            <p-table
                                #dataTable
                                sortMode="multiple"
                                (onLazyLoad)="getPeople($event)"
                                [value]="primengTableHelper.records"
                                rows="{{ primengTableHelper.defaultRecordsCountPerPage }}"
                                [paginator]="false"
                                [lazy]="true"
                                [scrollable]="true"
                                ScrollWidth="100%"
                                scrollDirection="horizontal"
                                [responsive]="primengTableHelper.isResponsive"
                                [resizableColumns]="primengTableHelper.resizableColumns"
                            >
                                <ng-template pTemplate="header">
                                    <tr>
                                        <th pSortableColumn="name">
                                            {{ 'Name' | localize }}
                                            <p-sortIcon field="name"></p-sortIcon>
                                        </th>
                                        <th pSortableColumn="surname">
                                            {{ 'Surname' | localize }}
                                            <p-sortIcon field="surname"></p-sortIcon>
                                        </th>
                                        <th pSortableColumn="emailAddress">
                                            {{ 'EmailAddress' | localize }}
                                            <p-sortIcon field="emailAddress"></p-sortIcon>
                                        </th>
                                    </tr>
                                </ng-template>
                                <ng-template pTemplate="body" let-record="$implicit">
                                    <tr>
                                        <td style="width: 150px">
                                            <span class="p-column-title">{{ 'FirstName' | localize }}</span>
                                            {{ record.name }}
                                        </td>
                                        <td style="width: 150px">
                                            <span class="p-column-title">{{ 'Surname' | localize }}</span>
                                            {{ record.surname }}
                                        </td>
                                        <td style="width: 250px">
                                            <span class="p-column-title">{{ 'EmailAddress' | localize }}</span>
                                            {{ record.emailAddress }}
                                        </td>
                                    </tr>
                                </ng-template>
                            </p-table>
                            <div class="primeng-no-data" *ngIf="primengTableHelper.totalRecordsCount == 0">
                                {{ 'NoData' | localize }}
                            </div>
                            <div class="primeng-paging-container">
                                <p-paginator
                                    [rows]="primengTableHelper.defaultRecordsCountPerPage"
                                    #paginator
                                    (onPageChange)="getPeople($event)"
                                    [totalRecords]="primengTableHelper.totalRecordsCount"
                                    [rowsPerPageOptions]="primengTableHelper.predefinedRecordsCountPerPage"
                                    [showCurrentPageReport]="true"
                                    [currentPageReportTemplate]="
                                        'TotalRecordsCount' | localize: primengTableHelper.totalRecordsCount
                                    "
                                ></p-paginator>
                            </div>
                        </div>
                        <!--<Primeng-TurboTable-End>-->
                    </div>
                </div>
            </div>
            <!---End Modal-->
            <createPersonModal #createPersonModal (modalSave)="getPeople()"></createPersonModal>
        </div>
    </div>
</div>

```

Made some minor changes in the view; Added a **button** to open the
modal and the **createPersonModal** component as like another HTML tag
(which matches to the selector in the
**create-person-modal.component.ts**).

## Next

- [Authorization For Phone Book](Developing-Step-By-Step-Angular-Authorization-PhoneBook)