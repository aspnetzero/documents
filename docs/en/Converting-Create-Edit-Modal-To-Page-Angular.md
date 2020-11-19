# Converting Create/Edit Modal to Page

In this document we will explain how to convert AspNet Zero's tenant create & edit modals to regular Angular pages.

## Create Tenant

### Create tenant page html

First, create a new html page named **create-tenant.component.html** with the content below. This is an empty page template for AspNet Zero's Angular pages.

````html
<div [@routerTransition]>
    <div class="kt-content  kt-grid__item kt-grid__item--fluid kt-grid kt-grid--hor">
        <div class="kt-subheader kt-grid__item">
            <div class="kt-container ">
                <div class="kt-subheader__main">
                    <h3 class="kt-subheader__title">
                        <span>{{"CreateNewTenant" | localize}}</span>
                    </h3>
                    <span class="kt-subheader__separator kt-subheader__separator--v"></span>
                    <span class="kt-subheader__desc">
                        {{"CreateTenantHeaderInfo" | localize}}
                    </span>
                </div>
            </div>
        </div>
        <div class="kt-container kt-grid__item kt-grid__item--fluid">
            <div class="kt-portlet kt-portlet--mobile">
                <div class="kt-portlet__body  kt-portlet__body--fit">
                    <p>PHONE BOOK CONTENT COMES HERE!</p>
                </div>
            </div>
        </div>
    </div>
</div>
````

After doing that, copy the form element from **create-tenant-modal.component.html** into div with "**kt-portlet__body**" class. Now, there are still modal related html code in our file, so we need to remove them.

First, remove the html item with **modal-header** class since we don't need it anymore.

Now, move all content of the div with class **modal-body** into the **form** tag. After that, we can remove the div with class **modal-body**. 

Finally, change the class of the div which contains Save and Cancel buttons from **modal-footer** to **kt-margin-t-40**.

Here is final version of **create-tenant.component.html**:

````html
<div [@routerTransition]>
    <div class="kt-content  kt-grid__item kt-grid__item--fluid kt-grid kt-grid--hor">
        <div class="kt-subheader kt-grid__item">
            <div class="kt-container ">
                <div class="kt-subheader__main">
                    <h3 class="kt-subheader__title">
                        <span>{{"CreateNewTenant" | localize}}</span>
                    </h3>
                    <span class="kt-subheader__separator kt-subheader__separator--v"></span>
                    <span class="kt-subheader__desc">
                        {{"CreateTenantHeaderInfo" | localize}}
                    </span>
                </div>
            </div>
        </div>
        <div class="kt-container kt-grid__item kt-grid__item--fluid">
            <div class="kt-portlet kt-portlet--mobile">
                <div class="kt-portlet__body  kt-portlet__body--fit">
                    <form #tenantCreateForm="ngForm" role="form" novalidate class="form-validation" *ngIf="active"
                        (submit)="save()">
                        <div class="form-group">
                            <label for="TenancyName">{{"TenancyName" | localize}} *</label>
                            <input id="TenancyName" #tenancyNameInput="ngModel" class="form-control" type="text"
                                [ngClass]="{'edited':tenant.tenancyName}" name="tenancyName"
                                [(ngModel)]="tenant.tenancyName" #tenancyName="ngModel" required maxlength="64"
                                pattern="^[a-zA-Z][a-zA-Z0-9_-]{1,}$">
                            <validation-messages [formCtrl]="tenancyNameInput"></validation-messages>
                        </div>
                        <div>
                            <span class="help-block text-danger"
                                *ngIf="!tenancyName.valid && !tenancyName.pristine">{{"TenantName_Regex_Description" | localize}}</span>
                        </div>

                        <div class="form-group">
                            <label for="Name">{{"TenantName" | localize}} *</label>
                            <input id="Name" #nameInput="ngModel" type="text" name="Name" class="form-control"
                                [ngClass]="{'edited':tenant.name}" [(ngModel)]="tenant.name" required maxlength="128">
                            <validation-messages [formCtrl]="nameInput"></validation-messages>
                        </div>

                        <div class="kt-checkbox-list">
                            <label class="kt-checkbox">
                                <input id="CreateTenant_UseHostDb" type="checkbox" name="UseHostDb"
                                    [(ngModel)]="useHostDb">
                                {{"UseHostDatabase" | localize}}
                                <span></span>
                            </label>
                        </div>

                        <div class="form-group" *ngIf="!useHostDb">
                            <label for="DatabaseConnectionString">{{"DatabaseConnectionString" | localize}} *</label>
                            <input id="DatabaseConnectionString" #connectionStringInput="ngModel" type="text"
                                name="ConnectionString" class="form-control" [(ngModel)]="tenant.connectionString"
                                [ngClass]="{'edited':tenant.connectionString}" required maxlength="1024">
                            <validation-messages [formCtrl]="connectionStringInput"></validation-messages>
                        </div>

                        <div class="form-group">
                            <label for="AdminEmailAddress">{{"AdminEmailAddress" | localize}} *</label>
                            <input id="AdminEmailAddress" #adminEmailAddressInput="ngModel" type="email"
                                name="AdminEmailAddress" class="form-control" [(ngModel)]="tenant.adminEmailAddress"
                                [ngClass]="{'edited':tenant.adminEmailAddress}" required
                                pattern="^\w+([-+.']\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$" maxlength="256">
                            <validation-messages [formCtrl]="adminEmailAddressInput"></validation-messages>
                        </div>

                        <div class="kt-checkbox-list">
                            <label class="kt-checkbox">
                                <input id="CreateTenant_SetRandomPassword" type="checkbox" name="SetRandomPassword"
                                    [(ngModel)]="setRandomPassword">
                                {{"SetRandomPassword" | localize}}
                                <span></span>
                            </label>
                        </div>

                        <div class="form-group" *ngIf="!setRandomPassword">
                            <label for="AdminPassword">{{"AdminPassword" | localize}}</label>
                            <input id="AdminPassword" type="password" name="adminPassword" class="form-control"
                                id="adminPassword" [(ngModel)]="tenant.adminPassword"
                                [ngClass]="{'edited':tenant.adminPassword}" [required]="!setRandomPassword"
                                #adminPassword="ngModel" validateEqual="adminPasswordRepeat" reverse="true"
                                maxlength="32" [requireDigit]="passwordComplexitySetting.requireDigit"
                                [requireLowercase]="passwordComplexitySetting.requireLowercase"
                                [requireUppercase]="passwordComplexitySetting.requireUppercase"
                                [requireNonAlphanumeric]="passwordComplexitySetting.requireNonAlphanumeric"
                                [requiredLength]="passwordComplexitySetting.requiredLength">
                        </div>

                        <div [hidden]="tenantCreateForm.form.valid || tenantCreateForm.form.pristine">
                            <ul class="help-block text-danger"
                                *ngIf="tenantCreateForm.controls['adminPassword'] && tenantCreateForm.controls['adminPassword'].errors">
                                <li [hidden]="!tenantCreateForm.controls['adminPassword'].errors.requireDigit">
                                    {{"PasswordComplexity_RequireDigit_Hint" | localize}}</li>
                                <li [hidden]="!tenantCreateForm.controls['adminPassword'].errors.requireLowercase">
                                    {{"PasswordComplexity_RequireLowercase_Hint" | localize}}</li>
                                <li [hidden]="!tenantCreateForm.controls['adminPassword'].errors.requireUppercase">
                                    {{"PasswordComplexity_RequireUppercase_Hint" | localize}}</li>
                                <li
                                    [hidden]="!tenantCreateForm.controls['adminPassword'].errors.requireNonAlphanumeric">
                                    {{"PasswordComplexity_RequireNonAlphanumeric_Hint" | localize}}</li>
                                <li [hidden]="!tenantCreateForm.controls['adminPassword'].errors.requiredLength">
                                    {{"PasswordComplexity_RequiredLength_Hint" | localize:passwordComplexitySetting.requiredLength}}
                                </li>
                            </ul>
                        </div>

                        <div class="form-group" *ngIf="!setRandomPassword">
                            <label for="AdminPasswordRepeat">{{"AdminPasswordRepeat" | localize}}</label>
                            <input id="AdminPasswordRepeat" type="password" name="adminPasswordRepeat"
                                class="form-control" [(ngModel)]="tenant.adminPasswordRepeat"
                                [ngClass]="{'edited':tenant.adminPasswordRepeat}" [required]="!setRandomPassword"
                                #adminPasswordRepeat="ngModel" [requireDigit]="passwordComplexitySetting.requireDigit"
                                [requireLowercase]="passwordComplexitySetting.requireLowercase"
                                [requireUppercase]="passwordComplexitySetting.requireUppercase"
                                [requireNonAlphanumeric]="passwordComplexitySetting.requireNonAlphanumeric"
                                [requiredLength]="passwordComplexitySetting.requiredLength"
                                validateEqual="adminPassword" maxlength="32">
                        </div>

                        <div [hidden]="tenantCreateForm.form.valid || tenantCreateForm.form.pristine">
                            <ul class="help-block text-danger"
                                *ngIf="tenantCreateForm.controls['adminPasswordRepeat'] && tenantCreateForm.controls['adminPasswordRepeat'].errors">
                                <li [hidden]="!tenantCreateForm.controls['adminPasswordRepeat'].errors.requireDigit">
                                    {{"PasswordComplexity_RequireDigit_Hint" | localize}}</li>
                                <li
                                    [hidden]="!tenantCreateForm.controls['adminPasswordRepeat'].errors.requireLowercase">
                                    {{"PasswordComplexity_RequireLowercase_Hint" | localize}}</li>
                                <li
                                    [hidden]="!tenantCreateForm.controls['adminPasswordRepeat'].errors.requireUppercase">
                                    {{"PasswordComplexity_RequireUppercase_Hint" | localize}}</li>
                                <li
                                    [hidden]="!tenantCreateForm.controls['adminPasswordRepeat'].errors.requireNonAlphanumeric">
                                    {{"PasswordComplexity_RequireNonAlphanumeric_Hint" | localize}}</li>
                                <li [hidden]="!tenantCreateForm.controls['adminPasswordRepeat'].errors.requiredLength">
                                    {{"PasswordComplexity_RequiredLength_Hint" | localize:passwordComplexitySetting.requiredLength}}
                                </li>
                                <li [hidden]="tenantCreateForm.controls['adminPasswordRepeat'].valid">
                                    {{"PasswordsDontMatch" | localize}}</li>
                            </ul>
                        </div>

                        <div class="form-group">
                            <label for="edition">{{"Edition" | localize}}</label>
                            <select id="edition" name="edition" class="form-control" [(ngModel)]="tenant.editionId"
                                (change)="onEditionChange($event)">
                                <option *ngFor="let edition of editions" [value]="edition.value">{{edition.displayText}}
                                </option>
                            </select>
                        </div>

                        <div [hidden]="!isSubscriptionFieldsVisible" class="kt-checkbox-list">
                            <label for="CreateTenant_IsUnlimited" class="kt-checkbox">
                                <input id="CreateTenant_IsUnlimited" type="checkbox" name="IsUnlimited"
                                    [(ngModel)]="isUnlimited" />
                                {{"UnlimitedTimeSubscription" | localize}}
                                <span></span>
                            </label>
                        </div>

                        <div [hidden]="isUnlimited || !isSubscriptionFieldsVisible" class="form-group"
                            [ngClass]="{'has-error': !subscriptionEndDateIsValid()}">
                            <label for="SubscriptionEndDate">{{"SubscriptionEndDate" | localize}}</label>
                            <input id="SubscriptionEndDate" type="text" #SubscriptionEndDateUtc
                                name="SubscriptionEndDateUtc" class="form-control" bsDatepicker
                                [(ngModel)]="tenant.subscriptionEndDateUtc" autocomplete="off">
                        </div>

                        <div [hidden]="!isSubscriptionFieldsVisible" class="kt-checkbox-list">
                            <label for="CreateTenant_IsInTrialPeriod" class="kt-checkbox">
                                <input id="CreateTenant_IsInTrialPeriod" type="checkbox" name="IsInTrialPeriod"
                                    [disabled]="isSelectedEditionFree" [(ngModel)]="tenant.isInTrialPeriod">
                                {{"IsInTrialPeriod" | localize}}
                                <span></span>
                            </label>
                        </div>

                        <div class="kt-checkbox-list">
                            <label for="CreateTenant_ShouldChangePasswordOnNextLogin" class="kt-checkbox">
                                <input id="CreateTenant_ShouldChangePasswordOnNextLogin" type="checkbox"
                                    name="ShouldChangePasswordOnNextLogin"
                                    [(ngModel)]="tenant.shouldChangePasswordOnNextLogin">
                                {{"ShouldChangePasswordOnNextLogin" | localize}}
                                <span></span>
                            </label>
                            <label for="CreateTenant_SendActivationEmail" class="kt-checkbox">
                                <input id="CreateTenant_SendActivationEmail" type="checkbox" name="SendActivationEmail"
                                    [(ngModel)]="tenant.sendActivationEmail">
                                {{"SendActivationEmail" | localize}}
                                <span></span>
                            </label>
                            <label for="CreateTenant_IsActive" class="kt-checkbox">
                                <input id="CreateTenant_IsActive" type="checkbox" name="IsActive"
                                    [(ngModel)]="tenant.isActive">
                                {{"Active" | localize}}
                                <span></span>
                            </label>
                        </div>
                        <div class="kt-margin-t-40">
                            <button type="button" [disabled]="saving" class="btn btn-secondary"
                                (click)="close()">{{"Cancel" | localize}}</button>
                            <button type="submit" [buttonBusy]="saving" [busyText]="l('SavingWithThreeDot')"
                                class="btn btn-primary"
                                [disabled]="!tenantCreateForm.form.valid || saving || !subscriptionEndDateIsValid()"><i
                                    class="fa fa-save"></i> <span>{{"Save" | localize}}</span></button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
````

### Create tenant page ts

Create a new file next to html file we have just created using the name **create-tenant.component.ts**. Copy the content of **create-tenant-modal.component.ts** to newly created file. Change the component name from `CreateTenantModalComponent` to `CreateTenantComponent`.

Modal page uses two methods named `show` and `onShown` which are not available in regular pages. So, we will implement `OnInit` and `AfterViewInit` in our Angular component and move the code of show method into `ngOnInit` and move the code of `onShown` into `ngAfterViewInit`. After doing that, we can delete empty `show` and `onShown` methods.

Import `OnInit` and `AfterViewInit` and move lines from `show` to `ngOnInit` and move lines from `onShown` to `ngAfterViewInit`.

Then, change the value of `templateUrl` from **./create-tenant-modal.component.html** to **./create-tenant.component.html**. You can also delete selector property of component definition since it is not mandatory. 

Since we are using animation when routing to create tenant page, import `appModuleAnimation` into the component and use it as the value for `animations` of the component definition.

````ts
// other imports.
import { appModuleAnimation } from '@shared/animations/routerTransition';

@Component({
    templateUrl: './create-tenant.component.html',
    animations: [appModuleAnimation()]
})
// component definition.
````


Also delete **modal** and **modalSave** properties since we don't need them anymore. Also, delete their usages in the component.

When the tenant is created, instead of closing the modal, we need to redirect user to tenant list. In order to do that, inject `Router` into our component and add below line into the close method of the component.

`this._router.navigate(['app/admin/tenants']);`

After all, you can delete **create-tenant-modal.component.ts** from your project.

Here is final version of **create-tenant.component.ts**:

````ts
import { Component, EventEmitter, Injector, Output, ViewChild, OnInit, AfterViewInit } from '@angular/core';
import { AppComponentBase } from '@shared/common/app-component-base';
import {
    CommonLookupServiceProxy, CreateTenantInput,
    PasswordComplexitySetting, ProfileServiceProxy,
    TenantServiceProxy, SubscribableEditionComboboxItemDto
} from '@shared/service-proxies/service-proxies';
import { filter as _filter } from 'lodash-es';
import { ModalDirective } from 'ngx-bootstrap';
import { finalize } from 'rxjs/operators';
import { appModuleAnimation } from '@shared/animations/routerTransition';
import { Router } from '@angular/router';

@Component({
    templateUrl: './create-tenant.component.html',
    animations: [appModuleAnimation()]
})
export class CreateTenantComponent extends AppComponentBase implements OnInit, AfterViewInit {

    active = false;
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

    ngAfterViewInit(): void {
        document.getElementById('TenancyName').focus();
    }

    ngOnInit(): void {
        this.active = true;
        this.init();

        this._profileService.getPasswordComplexitySetting().subscribe(result => {
            this.passwordComplexitySetting = result.setting;
        });
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
                    let defaultEdition = _filter(this.editions, { 'displayText': getDefaultEditionResult.name });
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
        let selectedEditions = _filter(this.editions, { 'value': this.tenant.editionId.toString() })
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
                this.close();
            });
    }

    close(): void {
        this.active = false;
        this._router.navigate(['app/admin/tenants']);
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
}
````

### Add route for tenant creation page

In order to navigate to create page, we need to add route to our route config. Add following path to `admin-routing.module.ts` for create tenant page.

````ts

...

import { CreateTenantComponent } from './tenants/create-tenant.component';

...

@NgModule({
    imports: [
        RouterModule.forChild([
            {
                path: '',
                children: [

                    ...

                    { path: 'tenants/create-tenant', component: CreateTenantComponent, data: { permission: 'Pages.Tenants.Create' } },

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

Remove following line from `tenants.component.html`.

````html
<createTenantModal #createTenantModal (modalSave)="getTenants()"></createTenantModal>
````

And following lines from `tenants.component.ts`

````ts
@ViewChild('createTenantModal') createTenantModal: CreateTenantModalComponent;
````

### Add CreateTenantComponent to AdminModule

In order to uses newly created CreateTenantComponent, we need to add it to our AdminModule.

Replace

`import { CreateTenantModalComponent } from './tenants/create-tenant-modal.component';`

with

`import { CreateTenantComponent } from './tenants/create-tenant.component';`

and use `CreateTenantComponent` instead of `CreateTenantModalComponent` in the imports list of AdminModule.

### Navigate to the tenant creation page

Import router into `tenants.component.ts`;

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

## Edit Tenant

### Edit tenant page html

First, create a new html page named **edit-tenant.component.html** with the content below. 

````html
<div [@routerTransition]>
    <div class="kt-content  kt-grid__item kt-grid__item--fluid kt-grid kt-grid--hor">
        <div class="kt-subheader kt-grid__item">
            <div class="kt-container ">
                <div class="kt-subheader__main">
                    <h3 class="kt-subheader__title">
                        <span>{{"EditTenant" | localize}}: {{tenant.tenancyName}}</span>
                    </h3>
                    <span class="kt-subheader__separator kt-subheader__separator--v"></span>
                    <span class="kt-subheader__desc">
                        {{"EditTenantHeaderInfo" | localize}}
                    </span>
                </div>
            </div>
        </div>
        <div class="kt-container kt-grid__item kt-grid__item--fluid">
            <div class="kt-portlet kt-portlet--mobile">
                <div class="kt-portlet__body  kt-portlet__body--fit">
                </div>
            </div>
        </div>
    </div>
</div>
````

After doing that, copy the form element from **edit-tenant-modal.component.html** into div with "**kt-portlet__body**" class. Now, there are still modal related html code in our file, so we need to remove them.

First, remove the html item with **modal-header** class since we don't need it anymore.

Now, move all content of the div with class **modal-body** into the **form** tag. After that, we can remove the div with class **modal-body**. 

Finally, change the class of the div which contains Save and Cancel buttons from **modal-footer** to **kt-margin-t-40**.

Here is final version of **edit-tenant.component.html**:

`````html
<div [@routerTransition]>
    <div class="kt-content  kt-grid__item kt-grid__item--fluid kt-grid kt-grid--hor">
        <div class="kt-subheader kt-grid__item">
            <div class="kt-container ">
                <div class="kt-subheader__main">
                    <h3 class="kt-subheader__title">
                        <span>{{"EditTenant" | localize}}: {{tenant.tenancyName}}</span>
                    </h3>
                    <span class="kt-subheader__separator kt-subheader__separator--v"></span>
                    <span class="kt-subheader__desc">
                        {{"EditTenantHeaderInfo" | localize}}
                    </span>
                </div>
            </div>
        </div>
        <div class="kt-container kt-grid__item kt-grid__item--fluid">
            <div class="kt-portlet kt-portlet--mobile">
                <div class="kt-portlet__body  kt-portlet__body--fit">
                    <form #tenantEditForm="ngForm" role="form" novalidate class="form-validation" (submit)="save()"
                        *ngIf="tenant && active">
                        <div class="form-group">
                            <label for="Name">{{"TenantName" | localize}} *</label>
                            <input id="Name" #nameInput="ngModel" type="text" name="Name" class="form-control"
                                [ngClass]="{'edited':tenant.name}" [(ngModel)]="tenant.name" required maxlength="128">
                            <validation-messages [formCtrl]="nameInput"></validation-messages>
                        </div>
                        <div class="form-group" *ngIf="currentConnectionString">
                            <label for="DatabaseConnectionString">{{"DatabaseConnectionString" | localize}} *</label>
                            <input id="DatabaseConnectionString" #connectionStringInput="ngModel" type="text"
                                name="ConnectionString" class="form-control" [(ngModel)]="tenant.connectionString"
                                required maxlength="1024">
                            <validation-messages [formCtrl]="connectionStringInput"></validation-messages>
                        </div>
                        <div *ngIf="currentConnectionString">
                            <span
                                class="help-block text-warning">{{"TenantDatabaseConnectionStringChangeWarningMessage" | localize}}</span>
                        </div>
                        <div class="form-group">
                            <label for="edition">{{"Edition" | localize}}</label>
                            <select id="edition" name="edition" class="form-control" [(ngModel)]="tenant.editionId"
                                (change)="onEditionChange($event)">
                                <option *ngFor="let edition of editions" [value]="edition.value">{{edition.displayText}}
                                </option>
                            </select>
                        </div>
                        <div [hidden]="!isSubscriptionFieldsVisible" class="kt-checkbox-list">
                            <label class="kt-checkbox">
                                <input id="CreateTenant_IsUnlimited" type="checkbox" name="IsUnlimited"
                                    [(ngModel)]="isUnlimited" (ngModelChange)="onUnlimitedChange()" />
                                {{"UnlimitedTimeSubscription" | localize}}
                                <span></span>
                            </label>
                        </div>
                        <div [hidden]="isUnlimited || !isSubscriptionFieldsVisible" class="form-group"
                            [ngClass]="{'has-error': !subscriptionEndDateUtcIsValid }">
                            <label for="SubscriptionEndDateUtc">{{"SubscriptionEndDateUtc" | localize}}</label>
                            <input id="SubscriptionEndDateUtc" type="datetime" #SubscriptionEndDateUtc
                                name="SubscriptionEndDateUtc" class="form-control"
                                [ngClass]="{'edited':tenant.subscriptionEndDateUtc}"
                                (bsValueChange)="subscriptionEndDateChange($event)" bsDatepicker
                                [(ngModel)]="tenant.subscriptionEndDateUtc" [required]="!isUnlimited">
                        </div>
                        <div [hidden]="!isSubscriptionFieldsVisible" class="kt-checkbox-list">
                            <label class="kt-checkbox">
                                <input id="CreateTenant_IsInTrialPeriod" type="checkbox" name="IsInTrialPeriod"
                                    [disabled]="selectedEditionIsFree()" [(ngModel)]="tenant.isInTrialPeriod">
                                {{"IsInTrialPeriod" | localize}}
                                <span></span>
                            </label>
                        </div>
                        <div class="kt-checkbox-list">
                            <label class="kt-checkbox">
                                <input id="EditTenant_IsActive" type="checkbox" name="IsActive"
                                    [(ngModel)]="tenant.isActive">
                                {{"Active" | localize}}
                                <span></span>
                            </label>
                        </div>
                        <div class="kt-margin-t-40">
                            <button type="button" [disabled]="saving" class="btn btn-secondary"
                                (click)="close()">{{"Cancel" | localize}}</button>
                            <button type="submit" [buttonBusy]="saving" [busyText]="l('SavingWithThreeDot')"
                                class="btn btn-primary"
                                [disabled]="!tenantEditForm.form.valid || saving || !subscriptionEndDateUtcIsValid"><i
                                    class="fa fa-save"></i> <span>{{"Save" | localize}}</span></button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
`````

### Edit tenant page ts

Create a new file next to html file we have just created using the name **edit-tenant.component.ts**. Copy the content of **edit-tenant-modal.component.ts** to newly created file. Change the component name from `EditTenantModalComponent` to `EditTenantComponent`.

Modal page uses two methods named `show` and `onShown` which are not available in regular pages. So, we will implement `OnInit` and `AfterViewInit` in our Angular component and move the code of show method into `ngOnInit` and move the code of `onShown` into `ngAfterViewInit`. After doing that, we can delete empty `show` and `onShown` methods.

Import `OnInit` and `AfterViewInit` and move lines from `show` to `ngOnInit` and move lines from `onShown` to `ngAfterViewInit`.

Then, change the value of `templateUrl` from **./edit-tenant-modal.component.html** to **./edit-tenant.component.html**. You can also delete selector property of component definition since it is not mandatory. 

Since we are using animation when routing to create tenant page, import `appModuleAnimation` into the component and use it as the value for `animations` of the component definition.

````ts
// other imports.
import { appModuleAnimation } from '@shared/animations/routerTransition';

@Component({
    templateUrl: './create-tenant.component.html',
    animations: [appModuleAnimation()]
})
// component definition.
````

`EditTenantModalComponent` was getting tenantId as a parameter of it's show method. We will be getting tenantId parameter from query string. In order to do this, first inject `ActivatedRoute` into `EditTenantComponent` like below;

````typescript
// other imports
import { ActivatedRoute } from '@angular/router';

@Component({
    templateUrl: './edit-tenant-modal.component.html',
    animations: [appModuleAnimation()]
})
export class EditTenantComponent extends AppComponentBase implements OnInit, AfterViewInit {

	// other code blocks...
    
    constructor(
        injector: Injector,
        private _tenantService: TenantServiceProxy,
        private _commonLookupService: CommonLookupServiceProxy,
        private _activatedRoute: ActivatedRoute
    ) {
        super(injector);
    }
	
	// other code blocks...
}	
````

and get it's value from query string in the ngOnInit like below;

````ts
var tenantId = this._activatedRoute.snapshot.queryParams['tenantId'];
````

Also delete tenantId parameter from ngOnInit method since we are now getting it from query parameters.

Also delete **modal** and **modalSave** properties since we don't need them anymore. Also, delete their usages in the component.

When the tenant is edited, instead of closing the modal, we need to redirect user to tenant list. In order to do that, inject `Router` into our component and add below line into the close method of the component.

`this._router.navigate(['app/admin/tenants']);`

After all, you can delete **edit-tenant-modal.component.ts** from your project.

Here is final version of **edit-tenant.component.ts**:

````ts
import { Component, ElementRef, EventEmitter, Injector, Output, ViewChild, OnInit, AfterViewInit } from '@angular/core';
import { AppComponentBase } from '@shared/common/app-component-base';
import { CommonLookupServiceProxy, SubscribableEditionComboboxItemDto, TenantEditDto, TenantServiceProxy } from '@shared/service-proxies/service-proxies';
import { filter as _filter } from 'lodash-es';;
import { DateTime } from 'luxon';
import { ModalDirective } from 'ngx-bootstrap';
import { finalize } from 'rxjs/operators';
import { appModuleAnimation } from '@shared/animations/routerTransition';
import { ActivatedRoute, Router } from '@angular/router';
import { DateTimeService } from '@app/shared/common/timing/date-time.service';

@Component({
    templateUrl: './edit-tenant.component.html',
    animations: [appModuleAnimation()]
})
export class EditTenantComponent extends AppComponentBase implements OnInit, AfterViewInit {

    @ViewChild('nameInput') nameInput: ElementRef;
    @ViewChild('SubscriptionEndDateUtc') subscriptionEndDateUtc: ElementRef;

    active = false;
    saving = false;
    isUnlimited = false;
    subscriptionEndDateUtcIsValid = false;
    subscriptionEndDateUtcx: DateTime;

    tenant: TenantEditDto = undefined;
    currentConnectionString: string;
    editions: SubscribableEditionComboboxItemDto[] = [];
    isSubscriptionFieldsVisible = false;

    constructor(
        injector: Injector,
        private _tenantService: TenantServiceProxy,
        private _commonLookupService: CommonLookupServiceProxy,
        private _activatedRoute: ActivatedRoute,
        private _router: Router,
        private _dateTimeService: DateTimeService
    ) {
        super(injector);
    }

    ngOnInit(): void {
        this.subscriptionEndDateUtcx = _dateTimeService.getStartOfDay();
        
        this.active = true;

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

        let selectedEditions = _filter(this.editions, { value: this.tenant.editionId + '' });
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
                this.close();
            });
    }

    close(): void {
        this.active = false;
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


### Add route for tenant edit page

Add following path to `admin-routing.module.ts` for edit tenant page.

````ts

...

import { EditTenantComponent } from './tenants/edit-tenant.component';

...

@NgModule({
    imports: [
        RouterModule.forChild([
            {
                path: '',
                children: [

                    ...

                    { path: 'tenants/edit-tenant', component: EditTenantComponent, data: { permission: 'Pages.Tenants.Edit' } },

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

Remove following lines from `tenants.component.ts`;

````typescript
import { EditTenantModalComponent } from './edit-tenant-modal.component';
````

````typescript
@ViewChild('editTenantModal') editTenantModal: EditTenantModalComponent;
````

### Add CreateTenantComponent to AdminModule

In order to uses newly created EditTenantComponent, we need to add it to our AdminModule.

Replace

`import { EditTenantModalComponent } from './tenants/edit-tenant-modal.component';`

with

`import { EditTenantComponent } from './tenants/edit-tenant.component';`

and use `EditTenantComponent` instead of `EditTenantModalComponent` in the imports list of AdminModule.

### Navigate to the tenant edit page

Router was already injected into TenantsComponent when converting create modal to a page. We can use it to navigate to tenant edit page. In order to do that, create a new method named `editTenant` in tenants.component.ts like below;

````ts
editTenant(tenantId): void {
        this._router.navigate(['app/admin/tenants/edit-tenant'], { queryParams: { tenantId: tenantId } });
    }
````

And use it in tenants.component.html like below instead of `editTenantModal.show(record.id)`;

````typescript
 <a href="javascript:;" *ngIf="permission.isGranted('Pages.Tenants.Edit')"
                                                       (click)="editTenant(record.id)">{{'Edit' | localize}}</a>
````

That's all. Now we can run the project, open tenant list and create a new tenant or edit an existing tenant in a new page instead of modal window.
