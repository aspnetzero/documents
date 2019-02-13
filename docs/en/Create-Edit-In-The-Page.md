# Create And Edit In The Page Without Popup Modal

You can change "create and edit" modals to full pages. In this article, we will explain how to do this for "create/edit" tenant modals.

## Create Tenant

### Add route for tenant creation page

Add following path to `admin-routing.module.ts` for create tenant page.

````ts

...

import { CreateTenantModalComponent } from './tenants/create-tenant-modal.component';

...

@NgModule({
    imports: [
        RouterModule.forChild([
            {
                path: '',
                children: [

                    ...

                    { path: 'tenants/create-tenant', component: CreateTenantModalComponent, data: { permission: 'Pages.Tenants.Create' } },

                    ...

                ]
            }
        ])
    ],
    exports: [
        RouterModule
    ]
})
...
````

### Remove createTenantModal component

Remove following line `tenants.component.html`.

````html
<createTenantModal #createTenantModal (modalSave)="getTenants()"></createTenantModal>
````

And following line from `tenants.component.ts`

````ts
@ViewChild('createTenantModal') createTenantModal: CreateTenantModalComponent;
````

### Navigate to the tenant creation page

Import router.

````ts
import { ActivatedRoute, Router } from '@angular/router';

constructor(
        ...
        private _router: Router,
        ...
    ) {
        ...
    }
````

And navigate to tenant creation page.

````ts
createTenant(): void {
        this._router.navigate(['app/admin/tenants/create-tenant']);
}
````

### Create tenant page html

Remove all modal related html elements and use html elements like other pages. There is no need to change any form and form elements. Latest tenant creation page look like following:

````html
<div [@routerTransition]>
    <div class="m-subheader">
        <div class="row align-items-center">
            <div class="mr-auto col-auto">
                <h3 class="m-subheader__title m-subheader__title--separator">
                    <span>{{"CreateNewTenant" | localize}}</span>
                </h3>
                <span class="m-section__sub">
                    {{"CreateTenantHeaderInfo" | localize}}
                </span>
            </div>
        </div>
    </div>
    <div class="m-content">
        <div class="m-portlet m-portlet--mobile">
            <div class="m-portlet__body">
                <form #tenantCreateForm="ngForm" role="form" novalidate class="form-validation" (submit)="save()">
                    <div class="form-group">
                        <label for="TenancyName">{{"TenancyName" | localize}} *</label>
                        <input id="TenancyName" #tenancyNameInput="ngModel" class="form-control" type="text" [ngClass]="{'edited':tenant.tenancyName}" name="tenancyName" [(ngModel)]="tenant.tenancyName" #tenancyName="ngModel" required maxlength="64" pattern="^[a-zA-Z][a-zA-Z0-9_-]{1,}$">
                        <validation-messages [formCtrl]="tenancyNameInput"></validation-messages>
                    </div>
                    <div>
                        <span class="help-block text-danger" *ngIf="!tenancyName.valid && !tenancyName.pristine">{{"TenantName_Regex_Description" | localize}}</span>
                    </div>
                    <div class="form-group">
                        <label for="Name">{{"TenantName" | localize}} *</label>
                        <input id="Name" #nameInput="ngModel" type="text" name="Name" class="form-control" [ngClass]="{'edited':tenant.name}" [(ngModel)]="tenant.name" required maxlength="128">
                        <validation-messages [formCtrl]="nameInput"></validation-messages>
                    </div>
                    <div class="m-checkbox-list">
                        <label class="m-checkbox">
                            <input id="CreateTenant_UseHostDb" type="checkbox" name="UseHostDb" [(ngModel)]="useHostDb">
                            {{"UseHostDatabase" | localize}}
                            <span></span>
                        </label>
                    </div>
                    <div class="form-group" *ngIf="!useHostDb">
                        <label for="DatabaseConnectionString">{{"DatabaseConnectionString" | localize}} *</label>
                        <input id="DatabaseConnectionString" #connectionStringInput="ngModel" type="text" name="ConnectionString" class="form-control" [(ngModel)]="tenant.connectionString" [ngClass]="{'edited':tenant.connectionString}" required maxlength="1024">
                        <validation-messages [formCtrl]="connectionStringInput"></validation-messages>
                    </div>
                    <div class="form-group">
                        <label for="AdminEmailAddress">{{"AdminEmailAddress" | localize}} *</label>
                        <input id="AdminEmailAddress" #adminEmailAddressInput="ngModel" type="email" name="AdminEmailAddress" class="form-control" [(ngModel)]="tenant.adminEmailAddress" [ngClass]="{'edited':tenant.adminEmailAddress}" required pattern="^\w+([-+.']\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$" maxlength="256">
                        <validation-messages [formCtrl]="adminEmailAddressInput"></validation-messages>
                    </div>
                    <div class="m-checkbox-list">
                        <label class="m-checkbox">
                            <input id="CreateTenant_SetRandomPassword" type="checkbox" name="SetRandomPassword" [(ngModel)]="setRandomPassword">
                            {{"SetRandomPassword" | localize}}
                            <span></span>
                        </label>
                    </div>
                    <div class="form-group" *ngIf="!setRandomPassword">
                        <label for="AdminPassword">{{"AdminPassword" | localize}}</label>
                        <input id="AdminPassword" type="password" name="adminPassword" class="form-control" id="adminPassword"
                               [(ngModel)]="tenant.adminPassword" [ngClass]="{'edited':tenant.adminPassword}" [required]="!setRandomPassword"
                               #adminPassword="ngModel" validateEqual="adminPasswordRepeat" reverse="true" maxlength="32" [requireDigit]="passwordComplexitySetting.requireDigit" [requireLowercase]="passwordComplexitySetting.requireLowercase"
                               [requireUppercase]="passwordComplexitySetting.requireUppercase" [requireNonAlphanumeric]="passwordComplexitySetting.requireNonAlphanumeric" [requiredLength]="passwordComplexitySetting.requiredLength">
                    </div>
                    <div [hidden]="tenantCreateForm.form.valid || tenantCreateForm.form.pristine">
                        <ul class="help-block text-danger" *ngIf="tenantCreateForm.controls['adminPassword'] && tenantCreateForm.controls['adminPassword'].errors">
                            <li [hidden]="!tenantCreateForm.controls['adminPassword'].errors.requireDigit">{{"PasswordComplexity_RequireDigit_Hint" | localize}}</li>
                            <li [hidden]="!tenantCreateForm.controls['adminPassword'].errors.requireLowercase">{{"PasswordComplexity_RequireLowercase_Hint" | localize}}</li>
                            <li [hidden]="!tenantCreateForm.controls['adminPassword'].errors.requireUppercase">{{"PasswordComplexity_RequireUppercase_Hint" | localize}}</li>
                            <li [hidden]="!tenantCreateForm.controls['adminPassword'].errors.requireNonAlphanumeric">{{"PasswordComplexity_RequireNonAlphanumeric_Hint" | localize}}</li>
                            <li [hidden]="!tenantCreateForm.controls['adminPassword'].errors.requiredLength">{{"PasswordComplexity_RequiredLength_Hint" | localize:passwordComplexitySetting.requiredLength}}</li>
                        </ul>
                    </div>
                    <div class="form-group" *ngIf="!setRandomPassword">
                        <label for="AdminPasswordRepeat">{{"AdminPasswordRepeat" | localize}}</label>
                        <input id="AdminPasswordRepeat" type="password" name="adminPasswordRepeat" class="form-control"
                               [(ngModel)]="tenant.adminPasswordRepeat" [ngClass]="{'edited':tenant.adminPasswordRepeat}" [required]="!setRandomPassword"
                               #adminPasswordRepeat="ngModel" [requireDigit]="passwordComplexitySetting.requireDigit" [requireLowercase]="passwordComplexitySetting.requireLowercase"
                               [requireUppercase]="passwordComplexitySetting.requireUppercase" [requireNonAlphanumeric]="passwordComplexitySetting.requireNonAlphanumeric" [requiredLength]="passwordComplexitySetting.requiredLength"
                               validateEqual="adminPassword"
                               maxlength="32">
                    </div>
                    <div [hidden]="tenantCreateForm.form.valid || tenantCreateForm.form.pristine">
                        <ul class="help-block text-danger" *ngIf="tenantCreateForm.controls['adminPasswordRepeat'] && tenantCreateForm.controls['adminPasswordRepeat'].errors">
                            <li [hidden]="!tenantCreateForm.controls['adminPasswordRepeat'].errors.requireDigit">{{"PasswordComplexity_RequireDigit_Hint" | localize}}</li>
                            <li [hidden]="!tenantCreateForm.controls['adminPasswordRepeat'].errors.requireLowercase">{{"PasswordComplexity_RequireLowercase_Hint" | localize}}</li>
                            <li [hidden]="!tenantCreateForm.controls['adminPasswordRepeat'].errors.requireUppercase">{{"PasswordComplexity_RequireUppercase_Hint" | localize}}</li>
                            <li [hidden]="!tenantCreateForm.controls['adminPasswordRepeat'].errors.requireNonAlphanumeric">{{"PasswordComplexity_RequireNonAlphanumeric_Hint" | localize}}</li>
                            <li [hidden]="!tenantCreateForm.controls['adminPasswordRepeat'].errors.requiredLength">{{"PasswordComplexity_RequiredLength_Hint" | localize:passwordComplexitySetting.requiredLength}}</li>
                            <li [hidden]="tenantCreateForm.controls['adminPasswordRepeat'].valid">{{"PasswordsDontMatch" | localize}}</li>
                        </ul>
                    </div>
                    <div class="form-group">
                        <label for="edition">{{"Edition" | localize}}</label>
                        <select id="edition" name="edition" class="form-control" [(ngModel)]="tenant.editionId" (change)="onEditionChange($event)">
                            <option *ngFor="let edition of editions" [value]="edition.value">{{edition.displayText}}</option>
                        </select>
                    </div>
                    <div [hidden]="!isSubscriptionFieldsVisible" class="m-checkbox-list">
                        <label for="CreateTenant_IsUnlimited" class="m-checkbox">
                            <input id="CreateTenant_IsUnlimited" type="checkbox" name="IsUnlimited" [(ngModel)]="isUnlimited" />
                            {{"UnlimitedTimeSubscription" | localize}}
                            <span></span>
                        </label>
                    </div>
                    <div [hidden]="isUnlimited || !isSubscriptionFieldsVisible" class="form-group" [ngClass]="{'has-error': !subscriptionEndDateIsValid()}">
                        <label for="SubscriptionEndDate">{{"SubscriptionEndDate" | localize}}</label>
                        <input id="SubscriptionEndDate" type="text" #SubscriptionEndDateUtc name="SubscriptionEndDateUtc" class="form-control" bsDatepicker [(ngModel)]="tenant.subscriptionEndDateUtc" autocomplete="off">
                    </div>
                    <div [hidden]="!isSubscriptionFieldsVisible" class="m-checkbox-list">
                        <label for="CreateTenant_IsInTrialPeriod" class="m-checkbox">
                            <input id="CreateTenant_IsInTrialPeriod" type="checkbox" name="IsInTrialPeriod" [disabled]="isSelectedEditionFree" [(ngModel)]="tenant.isInTrialPeriod">
                            {{"IsInTrialPeriod" | localize}}
                            <span></span>
                        </label>
                    </div>
                    <div class="m-checkbox-list">
                        <label for="CreateTenant_ShouldChangePasswordOnNextLogin" class="m-checkbox">
                            <input id="CreateTenant_ShouldChangePasswordOnNextLogin" type="checkbox" name="ShouldChangePasswordOnNextLogin" [(ngModel)]="tenant.shouldChangePasswordOnNextLogin">
                            {{"ShouldChangePasswordOnNextLogin" | localize}}
                            <span></span>
                        </label>
                        <label for="CreateTenant_SendActivationEmail" class="m-checkbox">
                            <input id="CreateTenant_SendActivationEmail" type="checkbox" name="SendActivationEmail" [(ngModel)]="tenant.sendActivationEmail">
                            {{"SendActivationEmail" | localize}}
                            <span></span>
                        </label>
                        <label for="CreateTenant_IsActive" class="m-checkbox">
                            <input id="CreateTenant_IsActive" type="checkbox" name="IsActive" [(ngModel)]="tenant.isActive">
                            {{"Active" | localize}}
                            <span></span>
                        </label>
                    </div>
                    <div class="m--margin-top-40">
                        <button type="button" [disabled]="saving" class="btn btn-secondary" (click)="cancel()">{{"Cancel" | localize}}</button>
                        <button type="submit" [buttonBusy]="saving" [busyText]="l('SavingWithThreeDot')" class="btn btn-primary" [disabled]="!tenantCreateForm.form.valid || saving || !subscriptionEndDateIsValid()"><i class="fa fa-save"></i> <span>{{"Save" | localize}}</span></button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
````

### Create tenant page ts

Import `OnInit` and `AfterViewInit` and move lines from `show` to `ngOnInit` and move lines from `onShown` to `ngAfterViewInit`.

Remove all modal related code. Latest tenant creation page ts looks:

````ts
import { Component, Injector, OnInit, AfterViewInit } from '@angular/core';
import { AppComponentBase } from '@shared/common/app-component-base';
import {
    CommonLookupServiceProxy, CreateTenantInput,
    PasswordComplexitySetting, ProfileServiceProxy,
    TenantServiceProxy, SubscribableEditionComboboxItemDto
} from '@shared/service-proxies/service-proxies';
import * as _ from 'lodash';
import { finalize } from 'rxjs/operators';
import {  Router } from '@angular/router';
import { appModuleAnimation } from '@shared/animations/routerTransition';

@Component({
    templateUrl: './create-tenant-modal.component.html',
    animations: [appModuleAnimation()]
})
export class CreateTenantModalComponent extends AppComponentBase implements OnInit, AfterViewInit {

    saving = false;
    setRandomPassword = true;
    useHostDb = true;
    editions: SubscribableEditionComboboxItemDto[] = [];
    tenant: CreateTenantInput;
    passwordComplexitySetting: PasswordComplexitySetting = new PasswordComplexitySetting();
    isUnlimited = false;
    isSubscriptionFieldsVisible = false;
    isSelectedEditionFree = false;

    constructor(
        injector: Injector,
        private _tenantService: TenantServiceProxy,
        private _commonLookupService: CommonLookupServiceProxy,
        private _profileService: ProfileServiceProxy,
        private _router: Router
    ) {
        super(injector);
    }

    ngOnInit(): void {
        this._profileService.getPasswordComplexitySetting().subscribe(result => {
            this.passwordComplexitySetting = result.setting;
        });

        this.init();
    }

    ngAfterViewInit(): void {
        document.getElementById('TenancyName').focus();
    }

    init(): void {
        this.tenant = new CreateTenantInput();
        this.tenant.isActive = true;
        this.tenant.shouldChangePasswordOnNextLogin = true;
        this.tenant.sendActivationEmail = true;
        this.tenant.editionId = 0;
        this.tenant.isInTrialPeriod = false;

        this._commonLookupService.getEditionsForCombobox(false)
            .subscribe((result) => {
                this.editions = result.items;

                let notAssignedItem = new SubscribableEditionComboboxItemDto();
                notAssignedItem.value = '';
                notAssignedItem.displayText = this.l('NotAssigned');

                this.editions.unshift(notAssignedItem);

                this._commonLookupService.getDefaultEditionName().subscribe((getDefaultEditionResult) => {
                    let defaultEdition = _.filter(this.editions, { 'displayText': getDefaultEditionResult.name });
                    if (defaultEdition && defaultEdition[0]) {
                        this.tenant.editionId = parseInt(defaultEdition[0].value);
                        this.toggleSubscriptionFields();
                    }
                });
            });
    }

    getEditionValue(item): number {
        return parseInt(item.value);
    }

    selectedEditionIsFree(): boolean {
        let selectedEditions = _.filter(this.editions, { 'value': this.tenant.editionId.toString() })
            .map(u => Object.assign(new SubscribableEditionComboboxItemDto(), u));

        if (selectedEditions.length !== 1) {
            this.isSelectedEditionFree = true;
        }

        let selectedEdition = selectedEditions[0];
        this.isSelectedEditionFree = selectedEdition.isFree;
        return this.isSelectedEditionFree;
    }

    subscriptionEndDateIsValid(): boolean {
        if (this.tenant.editionId <= 0) {
            return true;
        }

        if (this.isUnlimited) {
            return true;
        }

        if (!this.tenant.subscriptionEndDateUtc) {
            return false;
        }

        return this.tenant.subscriptionEndDateUtc !== undefined;
    }

    save(): void {
        this.saving = true;

        if (this.setRandomPassword) {
            this.tenant.adminPassword = null;
        }

        if (this.tenant.editionId === 0) {
            this.tenant.editionId = null;
        }

        if (this.isUnlimited || this.tenant.editionId <= 0) {
            this.tenant.subscriptionEndDateUtc = null;
        }

        this._tenantService.createTenant(this.tenant)
            .pipe(finalize(() => this.saving = false))
            .subscribe(() => {
                this.notify.info(this.l('SavedSuccessfully'));
                this._router.navigate(['app/admin/tenants']);
            });
    }

    onEditionChange(): void {
        this.tenant.isInTrialPeriod = this.tenant.editionId > 0 && !this.selectedEditionIsFree();
        this.toggleSubscriptionFields();
    }

    toggleSubscriptionFields() {
        this.isSelectedEditionFree = this.selectedEditionIsFree();
        if (this.tenant.editionId <= 0 || this.isSelectedEditionFree) {
            this.isSubscriptionFieldsVisible = false;

            if (this.isSelectedEditionFree) {
                this.isUnlimited = true;
            } else {
                this.isUnlimited = false;
            }
        } else {
            this.isSubscriptionFieldsVisible = true;
        }
    }

    cancel(): void {
        this._router.navigate(['app/admin/tenants']);
    }
}
````

## Edit Tenant

Tenant edit is the almost same like tenant creation. There is one different thing that you should pass tenant id to router.

### Add route for tenant edit page

Add following path to `admin-routing.module.ts` for edit tenant page.

````ts

...

import { EditTenantModalComponent } from './tenants/edit-tenant-modal.component';

...

@NgModule({
    imports: [
        RouterModule.forChild([
            {
                path: '',
                children: [

                    ...

                    { path: 'tenants/edit-tenant', component: EditTenantModalComponent, data: { permission: 'Pages.Tenants.Edit' } },

                    ...

                ]
            }
        ])
    ],
    exports: [
        RouterModule
    ]
})
...
````

### Remove editTenantModal component

Remove following line `tenants.component.html`.

````html
<editTenantModal #editTenantModal (modalSave)="getTenants()"></editTenantModal>
````

And following line from `tenants.component.ts`

````ts
@ViewChild('editTenantModal') editTenantModal: EditTenantModalComponent;
````

### Navigate to the tenant edit page

Import router.

````ts
import { ActivatedRoute, Router } from '@angular/router';

constructor(
        ...
        private _router: Router,
        ...
    ) {
        ...
    }
````

And navigate to tenant edit page.

````ts
editTenant(tenantId): void {
        this._router.navigate(['app/admin/tenants/edit-tenant'], { queryParams: { tenantId: tenantId } });
    }
````

### Edit tenant page html

Remove all modal related html elements and use html elements like other pages. There is no need to change any form and form elements. Latest tenant edit page look like following:

````html
<div [@routerTransition]>
    <div class="m-subheader">
        <div class="row align-items-center">
            <div class="mr-auto col-auto">
                <h3 class="m-subheader__title m-subheader__title--separator">
                    <span>{{"EditTenant" | localize}}: {{tenant.tenancyName}}</span>
                </h3>
                <span class="m-section__sub">
                    {{"EditTenantHeaderInfo" | localize}}
                </span>
            </div>
        </div>
    </div>
    <div class="m-content">
        <div class="m-portlet m-portlet--mobile">
            <div class="m-portlet__body">
                <form #tenantEditForm="ngForm" role="form" novalidate class="form-validation" (submit)="save()" *ngIf="tenant">
                    <div class="form-group">
                        <label for="Name">{{"TenantName" | localize}} *</label>
                        <input id="Name" #nameInput="ngModel" type="text" name="Name" class="form-control" [ngClass]="{'edited':tenant.name}" [(ngModel)]="tenant.name" required maxlength="128">
                        <validation-messages [formCtrl]="nameInput"></validation-messages>
                    </div>
                    <div class="form-group" *ngIf="currentConnectionString">
                        <label for="DatabaseConnectionString">{{"DatabaseConnectionString" | localize}} *</label>
                        <input id="DatabaseConnectionString" #connectionStringInput="ngModel" type="text" name="ConnectionString" class="form-control" [(ngModel)]="tenant.connectionString" required maxlength="1024">
                        <validation-messages [formCtrl]="connectionStringInput"></validation-messages>
                    </div>
                    <div *ngIf="currentConnectionString">
                        <span class="help-block text-warning">{{"TenantDatabaseConnectionStringChangeWarningMessage" | localize}}</span>
                    </div>
                    <div class="form-group">
                        <label for="edition">{{"Edition" | localize}}</label>
                        <select id="edition" name="edition" class="form-control" [(ngModel)]="tenant.editionId" (change)="onEditionChange($event)">
                            <option *ngFor="let edition of editions" [value]="edition.value">{{edition.displayText}}</option>
                        </select>
                    </div>
                    <div [hidden]="!isSubscriptionFieldsVisible" class="m-checkbox-list">
                        <label class="m-checkbox">
                            <input id="CreateTenant_IsUnlimited" type="checkbox" name="IsUnlimited" [(ngModel)]="isUnlimited" (ngModelChange)="onUnlimitedChange()" />
                            {{"UnlimitedTimeSubscription" | localize}}
                            <span></span>
                        </label>
                    </div>
                    <div [hidden]="isUnlimited || !isSubscriptionFieldsVisible" class="form-group" [ngClass]="{'has-error': !subscriptionEndDateUtcIsValid }">
                        <label for="SubscriptionEndDateUtc">{{"SubscriptionEndDateUtc" | localize}}</label>
                        <input id="SubscriptionEndDateUtc" type="datetime" #SubscriptionEndDateUtc name="SubscriptionEndDateUtc" class="form-control"
                               [ngClass]="{'edited':tenant.subscriptionEndDateUtc}"
                               (bsValueChange)="subscriptionEndDateChange($event)"
                               bsDatepicker
                               [(ngModel)]="tenant.subscriptionEndDateUtc"
                               [required]="!isUnlimited">
                    </div>
                    <div [hidden]="!isSubscriptionFieldsVisible" class="m-checkbox-list">
                        <label class="m-checkbox">
                            <input id="CreateTenant_IsInTrialPeriod" type="checkbox" name="IsInTrialPeriod" [disabled]="selectedEditionIsFree()" [(ngModel)]="tenant.isInTrialPeriod">
                            {{"IsInTrialPeriod" | localize}}
                            <span></span>
                        </label>
                    </div>
                    <div class="m-checkbox-list">
                        <label class="m-checkbox">
                            <input id="EditTenant_IsActive" type="checkbox" name="IsActive" [(ngModel)]="tenant.isActive">
                            {{"Active" | localize}}
                            <span></span>
                        </label>
                    </div>
                    <div class="m--margin-top-40">
                        <button type="button" [disabled]="saving" class="btn btn-secondary" (click)="cancel()">{{"Cancel" | localize}}</button>
                        <button type="submit" [buttonBusy]="saving" [busyText]="l('SavingWithThreeDot')" class="btn btn-primary" [disabled]="!tenantEditForm.form.valid || saving || !subscriptionEndDateUtcIsValid"><i class="fa fa-save"></i> <span>{{"Save" | localize}}</span></button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
````

### Edit tenant page ts

Import `OnInit` and `AfterViewInit` and move lines from `show` to `ngOnInit` and move lines from `onShown` to `ngAfterViewInit`.

Additionaly, you should get tenant id parameter in `ngOnInit` that is different than tenant creation page.

````ts
var tenantId = this._activatedRoute.snapshot.queryParams['tenantId'];
````

Remove all modal related code. And the latest tenant edit page ts looks:

````ts
import { Component, ElementRef, EventEmitter, Injector, Output, ViewChild, OnInit, AfterViewInit } from '@angular/core';
import { AppComponentBase } from '@shared/common/app-component-base';
import { CommonLookupServiceProxy, SubscribableEditionComboboxItemDto, TenantEditDto, TenantServiceProxy } from '@shared/service-proxies/service-proxies';
import * as _ from 'lodash';
import * as moment from 'moment';
import { ModalDirective } from 'ngx-bootstrap';
import { finalize } from 'rxjs/operators';
import { ActivatedRoute, Router } from '@angular/router';

@Component({
    selector: 'editTenantModal',
    templateUrl: './edit-tenant-modal.component.html'
})
export class EditTenantModalComponent extends AppComponentBase implements OnInit, AfterViewInit {

    @ViewChild('nameInput') nameInput: ElementRef;
    @ViewChild('editModal') modal: ModalDirective;
    @ViewChild('SubscriptionEndDateUtc') subscriptionEndDateUtc: ElementRef;

    @Output() modalSave: EventEmitter<any> = new EventEmitter<any>();

    saving = false;
    isUnlimited = false;
    subscriptionEndDateUtcIsValid = false;
    subscriptionEndDateUtcx: moment.Moment = moment().startOf('day');

    tenant: TenantEditDto = undefined;
    currentConnectionString: string;
    editions: SubscribableEditionComboboxItemDto[] = [];
    isSubscriptionFieldsVisible = false;

    constructor(
        injector: Injector,
        private _activatedRoute: ActivatedRoute,
        private _router: Router,
        private _tenantService: TenantServiceProxy,
        private _commonLookupService: CommonLookupServiceProxy
    ) {
        super(injector);
    }

    ngOnInit(): void {
        var tenantId = this._activatedRoute.snapshot.queryParams['tenantId'];

        this._commonLookupService.getEditionsForCombobox(false).subscribe(editionsResult => {
            this.editions = editionsResult.items;
            let notSelectedEdition = new SubscribableEditionComboboxItemDto();
            notSelectedEdition.displayText = this.l('NotAssigned');
            notSelectedEdition.value = '';
            this.editions.unshift(notSelectedEdition);

            this._tenantService.getTenantForEdit(tenantId).subscribe((tenantResult) => {
                this.tenant = tenantResult;
                this.currentConnectionString = tenantResult.connectionString;
                this.tenant.editionId = this.tenant.editionId || 0;
                this.isUnlimited = !this.tenant.subscriptionEndDateUtc;
                this.subscriptionEndDateUtcIsValid = this.isUnlimited || this.tenant.subscriptionEndDateUtc !== undefined;
                this.modal.show();
                this.toggleSubscriptionFields();
            });
        });
    }

    ngAfterViewInit(): void {
        document.getElementById('Name').focus();

        if (this.tenant.subscriptionEndDateUtc) {
            (this.subscriptionEndDateUtc.nativeElement as any).value = this.tenant.subscriptionEndDateUtc.format('L');
        }
    }

    subscriptionEndDateChange(e): void {
        this.subscriptionEndDateUtcIsValid = e && e.date !== false || this.isUnlimited;
    }

    selectedEditionIsFree(): boolean {
        if (!this.tenant.editionId) {
            return true;
        }

        let selectedEditions = _.filter(this.editions, { value: this.tenant.editionId + '' });
        if (selectedEditions.length !== 1) {
            return true;
        }

        let selectedEdition = selectedEditions[0];
        return selectedEdition.isFree;
    }

    save(): void {
        this.saving = true;
        if (this.tenant.editionId === 0) {
            this.tenant.editionId = null;
        }

        //take selected date as UTC
        if (this.isUnlimited || !this.tenant.editionId) {
            this.tenant.subscriptionEndDateUtc = null;
        }

        this._tenantService.updateTenant(this.tenant)
            .pipe(finalize(() => this.saving = false))
            .subscribe(() => {
                this.notify.info(this.l('SavedSuccessfully'));
                this._router.navigate(['app/admin/tenants']);
            });
    }

    cancel(): void {
        this._router.navigate(['app/admin/tenants']);
    }

    onEditionChange(): void {
        if (this.selectedEditionIsFree()) {
            this.tenant.isInTrialPeriod = false;
        }

        this.toggleSubscriptionFields();
    }

    onUnlimitedChange(): void {
        if (this.isUnlimited) {
            this.tenant.subscriptionEndDateUtc = null;
            this.subscriptionEndDateUtcIsValid = true;
        } else {
            if (!this.tenant.subscriptionEndDateUtc) {
                this.subscriptionEndDateUtcIsValid = false;
            }
        }
    }

    toggleSubscriptionFields() {
        if (this.tenant.editionId > 0) {
            this.isSubscriptionFieldsVisible = true;
        } else {
            this.isSubscriptionFieldsVisible = false;
        }
    }
}
````