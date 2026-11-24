# Converting Create/Edit Modal to Page

This document explains how to turn the tenant create and edit modals of the ASP.NET Zero Angular UI into regular pages. You can apply the same steps to any other create or edit modal.

> This document is written for v16.0 and later, where the Angular forms use Signal Forms. In v15.5 and earlier, the tenant modals use template-driven forms (`ngModel`) with the `validateEqual` and password complexity directives. The steps are the same for those versions: keep the form of your modal as it is and change only the modal-specific parts. To move a form to Signal Forms, see [Migrating to Signal Forms](Migrating-To-Signal-Forms-Angular.md).

## Create Tenant

### Create Tenant Page HTML

Create a file named **create-tenant.component.html** in the `src/app/admin/tenants` folder, next to **create-tenant-modal.component.html**, with the content below. This is the layout of an ASP.NET Zero page: a `sub-header` with the page title and breadcrumbs, and a card for the content.

```html
<div [@routerTransition]>
    <sub-header [title]="'CreateNewTenant' | localize" [breadcrumbs]="breadcrumbs" />

    <div [class]="containerClass">
        <div class="card card-custom gutter-b">
        </div>
    </div>
</div>
```

Copy the `form` element of **create-tenant-modal.component.html** into the `card` element. Then change the parts that belong to the modal:

1. Remove the `@if (active())` block around the form. A page shows its form as soon as it opens.
2. Remove the element with the `modal-header` class. The `sub-header` shows the title now.
3. Change the `modal-body` class to `card-body`.
4. Change the `modal-footer` class to `card-footer d-flex justify-content-end gap-3`.

Leave the fields as they are. They are bound with `[formField]`, and the rules of the form are declared in the component, including `passwordComplexity` and `mustMatch` for the admin password. The template needs no validation attributes.

Here is the final version of **create-tenant.component.html**:

```html
<div [@routerTransition]>
    <sub-header [title]="'CreateNewTenant' | localize" [breadcrumbs]="breadcrumbs" />

    <div [class]="containerClass">
        <div class="card card-custom gutter-b">
            <form [formRoot]="tenantForm" role="form" class="form-validation">
                <div class="card-body">
                    <div class="mb-5">
                        <label class="form-label required" for="TenancyName">{{ 'TenancyName' | localize }}</label>
                        <input
                            id="TenancyName"
                            class="form-control"
                            type="text"
                            [ngClass]="{ edited: tenant().tenancyName }"
                            [formField]="tenantForm.tenancyName" />
                        <validation-messages
                            [field]="tenantForm.tenancyName"
                            [customMessages]="{ pattern: 'TenancyName_Regex_Description' }" />
                    </div>
                    <div class="mb-5">
                        <label class="form-label required" for="Name">{{ 'Name' | localize }}</label>
                        <i
                            class="flaticon2-information icon-md text-muted ms-1"
                            data-bs-toggle="tooltip"
                            data-bs-placement="right"
                            [title]="'TenantNameTooltip' | localize"></i>
                        <input
                            id="Name"
                            type="text"
                            class="form-control"
                            [ngClass]="{ edited: tenant().name }"
                            [formField]="tenantForm.name" />
                        <validation-messages
                            [field]="tenantForm.name"
                            [customMessages]="{ pattern: 'Name_Regex_Description' }" />
                    </div>
                    <label class="form-check form-check-custom form-check-solid py-1 mb-5">
                        <input
                            #useHostDbInput
                            id="CreateTenant_UseHostDb"
                            type="checkbox"
                            [checked]="useHostDb()"
                            (change)="useHostDb.set(useHostDbInput.checked)"
                            class="form-check-input" />
                        <span class="form-check-label">
                            {{ 'UseHostDatabase' | localize }}
                        </span>
                    </label>
                    @if (!useHostDb()) {
                        <div class="mb-5">
                            <label class="form-label required" for="DatabaseConnectionString">
                                {{ 'DatabaseConnectionString' | localize }}
                            </label>
                            <input
                                id="DatabaseConnectionString"
                                type="text"
                                class="form-control"
                                [formField]="tenantForm.connectionString"
                                [ngClass]="{ edited: tenant().connectionString }" />
                            <validation-messages [field]="tenantForm.connectionString" />
                        </div>
                    }
                    <div class="mb-5">
                        <label class="form-label required" for="AdminName">{{ 'AdminName' | localize }}</label>
                        <input
                            id="AdminName"
                            type="text"
                            class="form-control"
                            [formField]="tenantForm.adminName"
                            [ngClass]="{ edited: tenant().adminName }" />
                        <validation-messages
                            [field]="tenantForm.adminName"
                            [customMessages]="{ pattern: 'AdminName_Regex_Description' }" />
                    </div>
                    <div class="mb-5">
                        <label class="form-label required" for="AdminSurname">
                            {{ 'AdminSurname' | localize }}
                        </label>
                        <input
                            id="AdminSurname"
                            type="text"
                            class="form-control"
                            [formField]="tenantForm.adminSurname"
                            [ngClass]="{ edited: tenant().adminSurname }" />
                        <validation-messages
                            [field]="tenantForm.adminSurname"
                            [customMessages]="{ pattern: 'AdminSurname_Regex_Description' }" />
                    </div>
                    <div class="mb-5">
                        <label class="form-label required" for="AdminEmailAddress">
                            {{ 'AdminEmailAddress' | localize }}
                        </label>
                        <input
                            id="AdminEmailAddress"
                            type="email"
                            class="form-control"
                            [formField]="tenantForm.adminEmailAddress"
                            [ngClass]="{ edited: tenant().adminEmailAddress }" />
                        <validation-messages
                            [field]="tenantForm.adminEmailAddress"
                            [customMessages]="{
                                pattern: 'AdminEmailAddress_Regex_Description',
                            }" />
                    </div>
                    <label class="form-check form-check-custom form-check-solid py-1 mb-5">
                        <input
                            #setRandomPasswordInput
                            id="CreateTenant_SetRandomPassword"
                            type="checkbox"
                            [checked]="setRandomPassword()"
                            (change)="setRandomPassword.set(setRandomPasswordInput.checked)"
                            class="form-check-input" />
                        <span class="form-check-label">
                            {{ 'SetRandomPassword' | localize }}
                        </span>
                    </label>
                    @if (!setRandomPassword()) {
                        <div class="mb-5">
                            <label class="form-label" for="adminPassword">{{ 'AdminPassword' | localize }}</label>
                            <input
                                id="adminPassword"
                                type="password"
                                class="form-control"
                                [formField]="tenantForm.adminPassword"
                                [ngClass]="{ edited: tenant().adminPassword }" />
                            <validation-messages
                                [field]="tenantForm.adminPassword"
                                [errorDefs]="passwordComplexityErrorDefs" />
                        </div>
                        <div class="mb-5">
                            <label for="AdminPasswordRepeat">{{ 'AdminPasswordRepeat' | localize }}</label>
                            <input
                                id="AdminPasswordRepeat"
                                type="password"
                                class="form-control"
                                [formField]="tenantForm.adminPasswordRepeat"
                                [ngClass]="{ edited: tenant().adminPasswordRepeat }" />
                            <validation-messages
                                [field]="tenantForm.adminPasswordRepeat"
                                [errorDefs]="passwordComplexityErrorDefs"
                                [customMessages]="{ mustMatch: 'PasswordsDontMatch' }" />
                        </div>
                    }
                    <div class="mb-5">
                        <label class="form-label" for="edition">{{ 'Edition' | localize }}</label>
                        <select
                            #editionSelect
                            id="edition"
                            class="form-select"
                            (change)="selectEdition(editionSelect.value)">
                            @for (edition of editions(); track edition.value) {
                                <option [value]="edition.value" [selected]="isEditionSelected(edition)">
                                    {{ edition.displayText }}
                                </option>
                            }
                        </select>
                    </div>
                    <label
                        class="form-check form-check-custom form-check-solid py-1"
                        [hidden]="!isSubscriptionFieldsVisible()">
                        <input
                            #isUnlimitedInput
                            id="CreateTenant_IsUnlimited"
                            type="checkbox"
                            [checked]="isUnlimited()"
                            (change)="changeIsUnlimited(isUnlimitedInput.checked)"
                            class="form-check-input" />
                        <span class="form-check-label">
                            {{ 'UnlimitedTimeSubscription' | localize }}
                        </span>
                    </label>
                    <div
                        [hidden]="isUnlimited() || !isSubscriptionFieldsVisible()"
                        class="mb-5 mt-5"
                        [ngClass]="{ 'has-error': tenantForm.subscriptionEndDateUtc().invalid() }">
                        <label for="SubscriptionEndDate">{{ 'SubscriptionEndDate' | localize }}</label>
                        <nz-date-picker
                            nzId="SubscriptionEndDate"
                            datePickerLuxonModifier
                            [date]="tenant().subscriptionEndDateUtc"
                            (dateChange)="tenantForm.subscriptionEndDateUtc().value.set($event)"
                            [isUtc]="true"
                            class="w-100" />
                    </div>
                    <label
                        class="form-check form-check-custom form-check-solid py-1"
                        [hidden]="isUnlimited() || !isSubscriptionFieldsVisible()">
                        <input
                            id="CreateTenant_IsInTrialPeriod"
                            type="checkbox"
                            [formField]="tenantForm.isInTrialPeriod"
                            class="form-check-input" />
                        <span class="form-check-label">
                            {{ 'IsInTrialPeriod' | localize }}
                        </span>
                    </label>
                    <label class="form-check form-check-custom form-check-solid py-1">
                        <input
                            id="CreateTenant_ShouldChangePasswordOnNextLogin"
                            type="checkbox"
                            [formField]="tenantForm.shouldChangePasswordOnNextLogin"
                            class="form-check-input" />
                        <span class="form-check-label">
                            {{ 'ShouldChangePasswordOnNextLogin' | localize }}
                        </span>
                    </label>
                    <label class="form-check form-check-custom form-check-solid py-1">
                        <input
                            id="CreateTenant_SendActivationEmail"
                            type="checkbox"
                            [formField]="tenantForm.sendActivationEmail"
                            class="form-check-input" />
                        <span class="form-check-label">
                            {{ 'SendActivationEmail' | localize }}
                        </span>
                    </label>
                    <label class="form-check form-check-custom form-check-solid py-1">
                        <input
                            id="CreateTenant_IsActive"
                            type="checkbox"
                            [formField]="tenantForm.isActive"
                            class="form-check-input" />
                        <span class="form-check-label">
                            {{ 'Active' | localize }}
                        </span>
                    </label>
                </div>
                <div class="card-footer d-flex justify-content-end gap-3">
                    <button type="button" [disabled]="saving()" class="btn btn-light-primary fw-bold" (click)="close()">
                        {{ 'Cancel' | localize }}
                    </button>
                    <button
                        type="submit"
                        [buttonBusy]="saving()"
                        [busyText]="l('SavingWithThreeDot')"
                        class="btn btn-primary fw-bold"
                        [disabled]="!tenantForm().valid() || saving()">
                        <i class="fa fa-save"></i>
                        <span>{{ 'Save' | localize }}</span>
                    </button>
                </div>
            </form>
        </div>
    </div>
</div>
```

### Create Tenant Page TypeScript

Copy **create-tenant-modal.component.ts** to a new file named **create-tenant.component.ts** in the same folder, and rename the class from `CreateTenantModalComponent` to `CreateTenantComponent`. Then make these changes:

1. **Component metadata:** Remove `selector`, because a routed page does not need one. Change `templateUrl` to `./create-tenant.component.html`, add `animations: [appModuleAnimation()]` for the router animation, and replace `AppBsModalDirective` with `SubHeaderComponent` in `imports`.
2. **Modal members:** Remove the `modal` view child, the `modalSave` output and the `active` signal. Remove the `ModalDirective` and `AppBsModalDirective` imports.
3. **Lifecycle:** A page has no `show` and `onShown` methods. Implement `OnInit` and `AfterViewInit`. Move the code of `show` into `ngOnInit` without the `this.modal()!.show()` call, and move the code of `onShown` into `ngAfterViewInit`. Then delete `show` and `onShown`.
4. **Initial values:** In `init`, remove the lines that reset the form and the signals. Every time the page opens, a new component instance is created with the initial values.
5. **Closing:** Inject `Router`, and change `close` to navigate back to the tenant list. In `save`, remove the `this.modalSave.emit(null)` call.
6. **Breadcrumbs:** Add the `breadcrumbs` property that the `sub-header` uses.

The form model, the form, its rules and the other methods stay as they are.

Here is the final version of **create-tenant.component.ts**:

```typescript
import { AfterViewInit, ChangeDetectionStrategy, Component, computed, inject, OnInit, signal } from '@angular/core';
import { Router } from '@angular/router';
import { disabled, form, FormField, FormRoot, hidden, maxLength, pattern, required } from '@angular/forms/signals';
import { AppComponentBase } from '@shared/common/app-component-base';
import {
    CommonLookupServiceProxy,
    CreateTenantInput,
    PasswordComplexitySetting,
    ProfileServiceProxy,
    TenantServiceProxy,
    SubscribableEditionComboboxItemDto,
} from '@shared/service-proxies/service-proxies';
import { filter as _filter } from 'lodash-es';
import { finalize } from 'rxjs/operators';
import { NgClass } from '@angular/common';
import { appModuleAnimation } from '@shared/animations/routerTransition';
import { BreadcrumbItem, SubHeaderComponent } from '@app/shared/common/sub-header/sub-header.component';
import { ValidationMessagesComponent } from '../../../shared/utils/validation-messages.component';
import {
    mustMatch,
    passwordComplexity,
    passwordComplexityErrorDefs,
} from '../../../shared/utils/validation/signal-form-validators';
import { NzDatePickerModule } from 'ng-zorro-antd/date-picker';
import { NzDatePickerLuxonModifierDirective } from '../../../shared/utils/date-time/nz-date-picker-luxon-modifier.directive';
import { ButtonBusyDirective } from '../../../shared/utils/button-busy.directive';
import { LocalizePipe } from '@shared/common/pipes/localize.pipe';
import { toFormModel } from '@shared/utils/forms/form-model';

const namePattern = /^[a-zA-Z][a-zA-Z0-9_-]{1,}$/;
const emailPattern = /^\w+([-+.']\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*$/;

const emptyTenantModel = () => ({
    ...toFormModel(new CreateTenantInput(), {
        tenancyName: '',
        name: '',
        adminName: '',
        adminSurname: '',
        adminEmailAddress: '',
        adminPassword: '',
        connectionString: '',
    }),
    isActive: true,
    shouldChangePasswordOnNextLogin: true,
    sendActivationEmail: true,
    isInTrialPeriod: false,
    adminPasswordRepeat: '',
});

const hasEdition = (editionId: number | null | undefined) => !!editionId && editionId > 0;

@Component({
    templateUrl: './create-tenant.component.html',
    animations: [appModuleAnimation()],
    changeDetection: ChangeDetectionStrategy.Eager,
    imports: [
        SubHeaderComponent,
        FormField,
        FormRoot,
        NgClass,
        ValidationMessagesComponent,
        NzDatePickerModule,
        NzDatePickerLuxonModifierDirective,
        ButtonBusyDirective,
        LocalizePipe,
    ],
})
export class CreateTenantComponent extends AppComponentBase implements OnInit, AfterViewInit {
    private _tenantService = inject(TenantServiceProxy);
    private _commonLookupService = inject(CommonLookupServiceProxy);
    private _profileService = inject(ProfileServiceProxy);
    private _router = inject(Router);

    readonly breadcrumbs = [
        new BreadcrumbItem(this.l('Tenants'), '/app/admin/tenants'),
        new BreadcrumbItem(this.l('CreateNewTenant')),
    ];

    readonly saving = signal(false);
    readonly setRandomPassword = signal(true);
    readonly useHostDb = signal(true);
    readonly editions = signal<SubscribableEditionComboboxItemDto[]>([]);
    readonly passwordComplexitySetting = signal<PasswordComplexitySetting>(new PasswordComplexitySetting());
    readonly passwordComplexityErrorDefs = passwordComplexityErrorDefs;
    readonly isUnlimited = signal(false);
    readonly isSubscriptionFieldsVisible = signal(false);
    readonly isSelectedEditionFree = signal(false);

    readonly tenant = signal(emptyTenantModel());

    readonly needsSubscriptionEndDate = computed(
        () => hasEdition(this.tenant().editionId) && this.isSubscriptionFieldsVisible() && !this.isUnlimited()
    );

    readonly tenantForm = form(
        this.tenant,
        (tenant) => {
            required(tenant.tenancyName);
            maxLength(tenant.tenancyName, 64);
            pattern(tenant.tenancyName, namePattern);

            required(tenant.name);
            maxLength(tenant.name, 128);
            pattern(tenant.name, namePattern);

            hidden(tenant.connectionString, () => this.useHostDb());
            required(tenant.connectionString);
            maxLength(tenant.connectionString, 1024);

            required(tenant.adminName);
            maxLength(tenant.adminName, 64);
            pattern(tenant.adminName, namePattern);

            required(tenant.adminSurname);
            maxLength(tenant.adminSurname, 64);
            pattern(tenant.adminSurname, namePattern);

            required(tenant.adminEmailAddress);
            maxLength(tenant.adminEmailAddress, 256);
            pattern(tenant.adminEmailAddress, emailPattern);

            hidden(tenant.adminPassword, () => this.setRandomPassword());
            required(tenant.adminPassword);
            maxLength(tenant.adminPassword, 32);
            passwordComplexity(tenant.adminPassword, () => this.passwordComplexitySetting());

            hidden(tenant.adminPasswordRepeat, () => this.setRandomPassword());
            required(tenant.adminPasswordRepeat);
            maxLength(tenant.adminPasswordRepeat, 32);
            passwordComplexity(tenant.adminPasswordRepeat, () => this.passwordComplexitySetting());
            mustMatch(tenant.adminPasswordRepeat, tenant.adminPassword);

            required(tenant.subscriptionEndDateUtc, { when: () => this.needsSubscriptionEndDate() });
            disabled(tenant.isInTrialPeriod, () => this.isSelectedEditionFree());
        },
        { submission: { action: async () => this.save() } }
    );

    ngOnInit(): void {
        this.init();

        this._profileService.getPasswordComplexitySetting().subscribe((result) => {
            this.passwordComplexitySetting.set(result.setting);
        });
    }

    ngAfterViewInit(): void {
        document.getElementById('TenancyName')!.focus();
    }

    init(): void {
        this._commonLookupService.getEditionsForCombobox(false).subscribe((result) => {
            const notAssignedItem = new SubscribableEditionComboboxItemDto();
            notAssignedItem.value = '';
            notAssignedItem.displayText = this.l('NotAssigned');
            this.editions.set([notAssignedItem, ...result.items!]);

            this._commonLookupService.getDefaultEditionName().subscribe((getDefaultEditionResult) => {
                const defaultEdition = _filter(this.editions(), { displayText: getDefaultEditionResult.name });
                if (defaultEdition && defaultEdition[0]) {
                    this.tenantForm.editionId().value.set(parseInt(defaultEdition[0].value!));
                    this.toggleSubscriptionFields();
                }
            });
        });
    }

    isEditionSelected(edition: SubscribableEditionComboboxItemDto): boolean {
        const editionId = this.tenant().editionId;
        return hasEdition(editionId) ? edition.value === editionId!.toString() : !edition.value;
    }

    selectEdition(value: string): void {
        this.tenantForm.editionId().value.set(value ? parseInt(value, 10) : null);
        this.onEditionChange();
    }

    selectedEditionIsFree(): boolean {
        const editionId = this.tenant().editionId;
        const selectedEditions = _filter(this.editions(), {
            value: hasEdition(editionId) ? editionId!.toString() : '',
        });
        return selectedEditions.length !== 1 || !!selectedEditions[0].isFree;
    }

    save(): void {
        const model = this.tenant();
        const editionSelected = hasEdition(model.editionId);
        const input = CreateTenantInput.fromJS(model);
        input.subscriptionEndDateUtc = model.subscriptionEndDateUtc ?? undefined;

        if (this.setRandomPassword()) {
            input.adminPassword = null!;
        }

        if (this.useHostDb()) {
            input.connectionString = null!;
        }

        if (!editionSelected) {
            input.editionId = null!;
        }

        if (this.isUnlimited()) {
            input.isInTrialPeriod = false;
        }

        if (this.isUnlimited() || !editionSelected) {
            input.subscriptionEndDateUtc = null!;
        }

        this.saving.set(true);
        this._tenantService
            .createTenant(input)
            .pipe(finalize(() => this.saving.set(false)))
            .subscribe(() => {
                this.notify.info(this.l('SavedSuccessfully'));
                this.close();
            });
    }

    close(): void {
        this._router.navigate(['/app/admin/tenants']);
    }

    onEditionChange(): void {
        this.tenantForm
            .isInTrialPeriod()
            .value.set(hasEdition(this.tenant().editionId) && !this.selectedEditionIsFree());
        this.toggleSubscriptionFields();
    }

    toggleSubscriptionFields() {
        const isFree = this.selectedEditionIsFree();
        this.isSelectedEditionFree.set(isFree);

        if (!hasEdition(this.tenant().editionId) || isFree) {
            this.isSubscriptionFieldsVisible.set(false);
            this.isUnlimited.set(isFree);
        } else {
            this.isSubscriptionFieldsVisible.set(true);
        }
    }

    changeIsUnlimited(isUnlimited: boolean) {
        this.isUnlimited.set(isUnlimited);
        if (isUnlimited) {
            this.tenantForm.isInTrialPeriod().value.set(false);
        }
    }
}
```

### Add a Route for the Create Page

Add a route for the page to the `admin` routes in **src/app/app.routes.ts**, after the `tenants` route:

```typescript
{
    path: 'tenants/create-tenant',
    loadComponent: () =>
        import('./admin/tenants/create-tenant.component').then((m) => m.CreateTenantComponent),
    data: { permission: 'Pages.Tenants.Create' },
},
```

The page is a standalone component that the route loads, so you do not register it in a module.

### Open the Create Page from the Tenant List

Remove the following line from **tenants.component.html**:

```html
<createTenantModal #createTenantModal (modalSave)="getTenants()" />
```

In **tenants.component.ts**, remove the `CreateTenantModalComponent` import, its entry in the `imports` of the component and the `createTenantModal` view child. `TenantsComponent` already injects `Router`, so change `createTenant` to navigate to the new page:

```typescript
createTenant(): void {
    this._router.navigate(['/app/admin/tenants/create-tenant']);
}
```

The tenant list loads its data in `ngOnInit`, so it shows the new tenant when the page navigates back to it.

Finally, delete **create-tenant-modal.component.ts**, **create-tenant-modal.component.html** and **create-tenant-modal.component.spec.ts**. If you move the tests of the modal to the page, add `provideNoopAnimations()` to the test providers and set a theme on `AppSessionService`, because the page uses the router animation and the `sub-header` reads the theme settings. **host-settings.component.spec.ts** shows how to set the theme.

## Edit Tenant

### Edit Tenant Page HTML

Create **edit-tenant.component.html** next to **edit-tenant-modal.component.html**, and copy the `form` element of the modal into the card in the same way as for the create page. Keep an `@if (loaded())` block around the form, because the form can be shown only after the tenant is loaded, and remove `active()` from its condition. The `sub-header` shows the tenancy name as its description.

Here is the final version of **edit-tenant.component.html**:

```html
<div [@routerTransition]>
    <sub-header
        [title]="'EditTenant' | localize"
        [description]="tenant().tenancyName"
        [breadcrumbs]="breadcrumbs" />

    <div [class]="containerClass">
        <div class="card card-custom gutter-b">
            @if (loaded()) {
                <form [formRoot]="tenantForm" role="form" class="form-validation">
                    <div class="card-body">
                        <div class="mb-5">
                            <label class="form-label required" for="Name">{{ 'TenantName' | localize }}</label>
                            <input
                                id="Name"
                                #nameInput
                                type="text"
                                class="form-control"
                                [ngClass]="{ edited: tenant().name }"
                                [formField]="tenantForm.name" />
                            <validation-messages
                                [field]="tenantForm.name"
                                [customMessages]="{ pattern: 'Name_Regex_Description' }" />
                        </div>
                        @if (currentConnectionString()) {
                            <div class="mb-5">
                                <label class="form-label required" for="DatabaseConnectionString">
                                    {{ 'DatabaseConnectionString' | localize }}
                                </label>
                                <input
                                    id="DatabaseConnectionString"
                                    type="text"
                                    class="form-control"
                                    [formField]="tenantForm.connectionString" />
                                <validation-messages [field]="tenantForm.connectionString" />
                            </div>
                            <div>
                                <span class="help-block text-warning">
                                    {{ 'TenantDatabaseConnectionStringChangeWarningMessage' | localize }}
                                </span>
                            </div>
                        }
                        <div class="mb-5">
                            <label class="form-label" for="edition">{{ 'Edition' | localize }}</label>
                            <select
                                #editionSelect
                                id="edition"
                                class="form-select"
                                (change)="selectEdition(editionSelect.value)">
                                @for (edition of editions(); track edition.value) {
                                    <option [value]="edition.value" [selected]="isEditionSelected(edition)">
                                        {{ edition.displayText }}
                                    </option>
                                }
                            </select>
                        </div>
                        <label
                            class="form-check form-check-custom form-check-solid py-1"
                            [hidden]="!isSubscriptionFieldsVisible()">
                            <input
                                #isUnlimitedInput
                                id="CreateTenant_IsUnlimited"
                                type="checkbox"
                                [checked]="isUnlimited()"
                                (change)="changeIsUnlimited(isUnlimitedInput.checked)"
                                class="form-check-input" />
                            <span class="form-check-label">
                                {{ 'UnlimitedTimeSubscription' | localize }}
                            </span>
                        </label>
                        <div
                            [hidden]="isUnlimited() || !isSubscriptionFieldsVisible()"
                            class="mb-5 mt-5"
                            [ngClass]="{ 'has-error': tenantForm.subscriptionEndDateUtc().invalid() }">
                            <label class="form-label" for="SubscriptionEndDateUtc">
                                {{ 'SubscriptionEndDateUtc' | localize }}
                            </label>
                            <nz-date-picker
                                nzId="SubscriptionEndDateUtc"
                                datePickerLuxonModifier
                                [date]="tenant().subscriptionEndDateUtc"
                                (dateChange)="tenantForm.subscriptionEndDateUtc().value.set($event)"
                                [isUtc]="true"
                                class="w-100" />
                        </div>
                        <label
                            class="form-check form-check-custom form-check-solid py-1"
                            [hidden]="isUnlimited() || !isSubscriptionFieldsVisible()">
                            <input
                                id="CreateTenant_IsInTrialPeriod"
                                type="checkbox"
                                [formField]="tenantForm.isInTrialPeriod"
                                class="form-check-input" />
                            <span class="form-check-label">
                                {{ 'IsInTrialPeriod' | localize }}
                            </span>
                        </label>
                        <label class="form-check form-check-custom form-check-solid py-1">
                            <input
                                id="EditTenant_IsActive"
                                type="checkbox"
                                [formField]="tenantForm.isActive"
                                class="form-check-input" />
                            <span class="form-check-label">
                                {{ 'Active' | localize }}
                            </span>
                        </label>
                    </div>
                    <div class="card-footer d-flex justify-content-end gap-3">
                        <button
                            type="button"
                            [disabled]="saving()"
                            class="btn btn-light-primary fw-bold"
                            (click)="close()">
                            {{ 'Cancel' | localize }}
                        </button>
                        <button
                            type="submit"
                            [buttonBusy]="saving()"
                            [busyText]="l('SavingWithThreeDot')"
                            class="btn btn-primary fw-bold"
                            [disabled]="!tenantForm().valid() || saving()">
                            <i class="fa fa-save"></i>
                            <span>{{ 'Save' | localize }}</span>
                        </button>
                    </div>
                </form>
            }
        </div>
    </div>
</div>
```

### Edit Tenant Page TypeScript

Copy **edit-tenant-modal.component.ts** to **edit-tenant.component.ts**, and rename the class from `EditTenantModalComponent` to `EditTenantComponent`. Make the same changes as for the create page to the component metadata, the modal members, `close`, `save` and the breadcrumbs. Remove `active`, but keep `loaded`.

Two parts are different. First, `show` received the tenant id as a parameter, and the page reads it from the `tenantId` query string parameter instead. Inject `ActivatedRoute`, move the code of `show` into `ngOnInit`, and read the id at the start of it:

```typescript
const tenantId = Number(this._activatedRoute.snapshot.queryParams['tenantId']);
```

Remove the `this.loaded.set(false)`, `this.active.set(true)` and `this.modal()!.show()` lines that you moved from `show`.

Second, the name input is rendered only after the tenant is loaded, so it does not exist yet when `ngAfterViewInit` runs. Inject `Injector`, and focus the input with `afterNextRender` after the `this.loaded.set(true)` line. Then delete `onShown`.

```typescript
afterNextRender(() => this.nameInput()?.nativeElement.focus(), { injector: this._injector });
```

Here is the final version of **edit-tenant.component.ts**:

```typescript
import {
    afterNextRender,
    ChangeDetectionStrategy,
    Component,
    computed,
    ElementRef,
    inject,
    Injector,
    OnInit,
    signal,
    viewChild,
} from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { disabled, form, FormField, FormRoot, hidden, maxLength, pattern, required } from '@angular/forms/signals';
import { AppComponentBase } from '@shared/common/app-component-base';
import {
    CommonLookupServiceProxy,
    SubscribableEditionComboboxItemDto,
    TenantEditDto,
    TenantServiceProxy,
} from '@shared/service-proxies/service-proxies';
import { filter as _filter } from 'lodash-es';
import { finalize, switchMap } from 'rxjs/operators';
import { NgClass } from '@angular/common';
import { appModuleAnimation } from '@shared/animations/routerTransition';
import { BreadcrumbItem, SubHeaderComponent } from '@app/shared/common/sub-header/sub-header.component';
import { ValidationMessagesComponent } from '../../../shared/utils/validation-messages.component';
import { NzDatePickerModule } from 'ng-zorro-antd/date-picker';
import { NzDatePickerLuxonModifierDirective } from '../../../shared/utils/date-time/nz-date-picker-luxon-modifier.directive';
import { ButtonBusyDirective } from '../../../shared/utils/button-busy.directive';
import { LocalizePipe } from '@shared/common/pipes/localize.pipe';
import { toFormModel } from '@shared/utils/forms/form-model';

const namePattern = /^[a-zA-Z][a-zA-Z0-9_-]{1,}$/;

const toTenantFormModel = (tenant: TenantEditDto) => toFormModel(tenant, { name: '', connectionString: '' });

const hasEdition = (editionId: number | null | undefined) => !!editionId && editionId > 0;

@Component({
    templateUrl: './edit-tenant.component.html',
    animations: [appModuleAnimation()],
    changeDetection: ChangeDetectionStrategy.Eager,
    imports: [
        SubHeaderComponent,
        FormField,
        FormRoot,
        NgClass,
        ValidationMessagesComponent,
        NzDatePickerModule,
        NzDatePickerLuxonModifierDirective,
        ButtonBusyDirective,
        LocalizePipe,
    ],
})
export class EditTenantComponent extends AppComponentBase implements OnInit {
    private _tenantService = inject(TenantServiceProxy);
    private _commonLookupService = inject(CommonLookupServiceProxy);
    private _activatedRoute = inject(ActivatedRoute);
    private _router = inject(Router);
    private _injector = inject(Injector);

    readonly nameInput = viewChild<ElementRef>('nameInput');

    readonly breadcrumbs = [
        new BreadcrumbItem(this.l('Tenants'), '/app/admin/tenants'),
        new BreadcrumbItem(this.l('EditTenant')),
    ];

    readonly saving = signal(false);
    readonly loaded = signal(false);
    readonly isUnlimited = signal(false);
    readonly currentConnectionString = signal<string | undefined>(undefined);
    readonly editions = signal<SubscribableEditionComboboxItemDto[]>([]);

    readonly tenant = signal(toTenantFormModel(new TenantEditDto()));

    readonly isSubscriptionFieldsVisible = computed(() => hasEdition(this.tenant().editionId));

    readonly tenantForm = form(
        this.tenant,
        (tenant) => {
            required(tenant.name);
            maxLength(tenant.name, 128);
            pattern(tenant.name, namePattern);

            hidden(tenant.connectionString, () => !this.currentConnectionString());
            required(tenant.connectionString);
            maxLength(tenant.connectionString, 1024);

            required(tenant.subscriptionEndDateUtc, {
                when: () => this.isSubscriptionFieldsVisible() && !this.isUnlimited(),
            });
            disabled(tenant.isInTrialPeriod, () => this.selectedEditionIsFree());
        },
        { submission: { action: async () => this.save() } }
    );

    ngOnInit(): void {
        const tenantId = Number(this._activatedRoute.snapshot.queryParams['tenantId']);

        this._commonLookupService
            .getEditionsForCombobox(false)
            .pipe(
                switchMap((editionsResult) => {
                    const notSelectedEdition = new SubscribableEditionComboboxItemDto();
                    notSelectedEdition.displayText = this.l('NotAssigned');
                    notSelectedEdition.value = '';
                    this.editions.set([notSelectedEdition, ...editionsResult.items!]);
                    return this._tenantService.getTenantForEdit(tenantId);
                })
            )
            .subscribe((tenantResult) => {
                this.currentConnectionString.set(tenantResult.connectionString);
                this.isUnlimited.set(!tenantResult.subscriptionEndDateUtc);
                this.tenantForm().reset(toTenantFormModel(tenantResult));
                this.loaded.set(true);
                afterNextRender(() => this.nameInput()?.nativeElement.focus(), { injector: this._injector });
            });
    }

    isEditionSelected(edition: SubscribableEditionComboboxItemDto): boolean {
        const editionId = this.tenant().editionId;
        return hasEdition(editionId) ? edition.value === editionId!.toString() : !edition.value;
    }

    selectEdition(value: string): void {
        this.tenantForm.editionId().value.set(value ? parseInt(value, 10) : null);
        if (this.selectedEditionIsFree()) {
            this.tenantForm.isInTrialPeriod().value.set(false);
        }
    }

    selectedEditionIsFree(): boolean {
        const editionId = this.tenant().editionId;
        if (!hasEdition(editionId)) {
            return true;
        }

        const selectedEditions = _filter(this.editions(), { value: editionId + '' });
        if (selectedEditions.length !== 1) {
            return true;
        }

        return !!selectedEditions[0].isFree;
    }

    changeIsUnlimited(isUnlimited: boolean): void {
        this.isUnlimited.set(isUnlimited);
        if (isUnlimited) {
            this.tenantForm.subscriptionEndDateUtc().value.set(null);
            this.tenantForm.isInTrialPeriod().value.set(false);
        }
    }

    save(): void {
        const model = this.tenant();
        const editionSelected = hasEdition(model.editionId);
        const input = TenantEditDto.fromJS(model);
        input.connectionString = model.connectionString || undefined;

        if (!editionSelected) {
            input.editionId = null!;
        }

        if (this.isUnlimited()) {
            input.isInTrialPeriod = false;
        }

        if (this.isUnlimited() || !editionSelected) {
            input.subscriptionEndDateUtc = null!;
        } else if (model.subscriptionEndDateUtc) {
            input.subscriptionEndDateUtc = model.subscriptionEndDateUtc.toLocal();
        }

        this.saving.set(true);
        this._tenantService
            .updateTenant(input)
            .pipe(finalize(() => this.saving.set(false)))
            .subscribe(() => {
                this.notify.info(this.l('SavedSuccessfully'));
                this.close();
            });
    }

    close(): void {
        this._router.navigate(['/app/admin/tenants']);
    }
}
```

### Add a Route for the Edit Page

Add the following route to the `admin` routes in **src/app/app.routes.ts**:

```typescript
{
    path: 'tenants/edit-tenant',
    loadComponent: () =>
        import('./admin/tenants/edit-tenant.component').then((m) => m.EditTenantComponent),
    data: { permission: 'Pages.Tenants.Edit' },
},
```

### Open the Edit Page from the Tenant List

Remove the following line from **tenants.component.html**:

```html
<editTenantModal #editTenantModal (modalSave)="getTenants()" />
```

In **tenants.component.ts**, remove the `EditTenantModalComponent` import, its entry in the `imports` of the component and the `editTenantModal` view child. The **Edit** action of a tenant is an item of the actions menu that **tenants.component.ts** builds. Change its `command` to navigate to the edit page with the id of the tenant:

```typescript
if (this.permission.isGranted('Pages.Tenants.Edit')) {
    items.push({
        label: this.l('Edit'),
        icon: 'fa fa-pencil-alt',
        command: () =>
            this._router.navigate(['/app/admin/tenants/edit-tenant'], {
                queryParams: { tenantId: tenant.id },
            }),
    });
}
```

Finally, delete **edit-tenant-modal.component.ts**, **edit-tenant-modal.component.html** and **edit-tenant-modal.component.spec.ts**.

Now you can open the tenant list and create or edit a tenant on its own page instead of in a modal.
