# Migrating to Signal Forms

Starting with v16.0, the Angular UI of ASP.NET Zero builds its forms with [Angular Signal Forms](https://angular.dev/guide/forms/signals/overview) (`@angular/forms/signals`) instead of template-driven forms (`ngModel`). This guide explains what changed in the solution, which changes can break your code when you upgrade, and how to move your own forms to Signal Forms.

Angular still supports template-driven forms. The forms you wrote yourself keep working after the upgrade unless they use one of the [removed directives](#breaking-changes), so you can migrate them at your own pace.

## What Changed

The create and edit modals, the account pages, the settings pages and the other form screens of the Angular UI use Signal Forms. Some controls stay on `ngModel`, because they are not forms that are validated and submitted:

- list page filters
- the internals of controls that implement `ControlValueAccessor`
- controls that only change the state of a page, for example a switch that reloads a list

The solution contains these shared helpers for Signal Forms:

| File | Contents |
|---|---|
| `src/shared/utils/forms/form-model.ts` | `toFormModel` builds a form model from an NSwag DTO. `fromFormModel` removes the empty values from a form model. |
| `src/shared/utils/validation/signal-form-validators.ts` | The `passwordComplexity` and `mustMatch` rules, and `passwordComplexityErrorDefs` for their messages. |
| `src/shared/utils/validation-messages.component.ts` | `<validation-messages>` shows the errors of a Signal Forms field with `[field]`. `[formCtrl]` still works for `ngModel`. |

Power Tools also generates Signal Forms code for v16.0 and later projects. See [Power Tools](#power-tools).

## Breaking Changes

| Removed or changed | Use instead |
|---|---|
| The `validateEqual` directive (`EqualValidator` in `src/shared/utils/validation/equal-validator.directive.ts`) | The `mustMatch(path, otherPath)` rule |
| The `requireDigit`, `requireLowercase`, `requireUppercase`, `requireNonAlphanumeric` and `requiredLength` directive (`PasswordComplexityValidator` in `src/shared/utils/validation/password-complexity-validator.directive.ts`) | The `passwordComplexity(path, () => setting)` rule |
| The `data` model of `password-input-with-show-button` is renamed to `value` | Bind it with `[formField]` in a form, or with `[(value)]` |

If one of your own template-driven forms uses a removed directive, migrate that form as described below. If you cannot migrate it yet, copy the directive file from your previous version into your project.

## Migrating a Form

The following create or edit modal uses a template-driven form:

```typescript
@Component({
    imports: [FormsModule, AppBsModalDirective, ValidationMessagesComponent, LocalizePipe],
})
export class CreateOrEditProductModalComponent extends AppComponentBase {
    product = new ProductEditDto();

    show(productId?: number): void {
        this._productService.getProductForEdit(productId).subscribe((result) => {
            this.product = result.product;
            this.active.set(true);
            this.modal().show();
        });
    }

    save(): void {
        this._productService.createOrUpdateProduct(this.product).subscribe(() => this.close());
    }
}
```

```html
<form #productForm="ngForm" novalidate (ngSubmit)="save()">
    <input id="ProductName" name="name" type="text" class="form-control"
           [(ngModel)]="product.name" #nameInput="ngModel" required maxlength="128" />
    <validation-messages [formCtrl]="nameInput"></validation-messages>

    <button type="submit" class="btn btn-primary" [disabled]="!productForm.form.valid">
        {{ 'Save' | localize }}
    </button>
</form>
```

The same modal with Signal Forms:

```typescript
import { form, FormField, FormRoot, maxLength, required } from '@angular/forms/signals';
import { toFormModel } from '@shared/utils/forms/form-model';

const toProductFormModel = (product: ProductEditDto) => toFormModel(product, { name: '', description: '' });

@Component({
    imports: [FormField, FormRoot, AppBsModalDirective, ValidationMessagesComponent, LocalizePipe],
})
export class CreateOrEditProductModalComponent extends AppComponentBase {
    readonly product = signal(toProductFormModel(new ProductEditDto()));

    readonly productForm = form(
        this.product,
        (product) => {
            required(product.name);
            maxLength(product.name, 128);
        },
        { submission: { action: async () => this.save() } }
    );

    show(productId?: number): void {
        this._productService.getProductForEdit(productId).subscribe((result) => {
            this.productForm().reset(toProductFormModel(result.product));
            this.active.set(true);
            this.modal().show();
        });
    }

    save(): void {
        const input = ProductEditDto.fromJS(this.product());
        this._productService.createOrUpdateProduct(input).subscribe(() => this.close());
    }
}
```

```html
<form [formRoot]="productForm">
    <input id="ProductName" type="text" class="form-control" [formField]="productForm.name" />
    <validation-messages [field]="productForm.name" />

    <button type="submit" class="btn btn-primary" [disabled]="!productForm().valid()">
        {{ 'Save' | localize }}
    </button>
</form>
```

To migrate a form:

1. **Build the form model with `toFormModel`.** Signal Forms copies the model on every change, so an NSwag DTO would lose its class and its `toJSON()` method after the first keystroke. `toFormModel` turns the DTO into a plain object, reads every property name from the DTO's `toJSON()` and turns `undefined` into `null`. Give every field that a text input, textarea or password input edits an empty string default, because a text input bound to `null` parses the typed text as a number.
2. **Declare the form and its rules.** Call `form(model, schema, { submission })` and move every validation attribute of the template into the schema. See [Rules Instead of Attributes](#rules-instead-of-attributes).
3. **Bind the template.** Connect the `form` element with `[formRoot]`, and bind each input with `[formField]`. Remove the `name`, `required`, `maxlength`, `pattern` and `[disabled]` attributes and the `#x="ngModel"` references: `[formField]` writes the matching native attributes for you.
4. **Show the messages.** Replace `<validation-messages [formCtrl]="...">` with `<validation-messages [field]="productForm.name" />`.
5. **Load and save.** Call `productForm().reset(...)` in `show()` before the form is displayed, so every open starts without validation messages. In `save()`, create the DTO with `ProductEditDto.fromJS(this.product())`. Never send the plain form model to a service proxy.
6. **Update the imports.** Import `FormField`, `FormRoot` and `ValidationMessagesComponent`, and remove `FormsModule` when nothing else in the component uses `ngModel`.

A nested DTO needs its own `toFormModel` call, because `toFormModel` normalizes one level:

```typescript
const toInstallFormModel = (dto: InstallDto) => ({
    ...toFormModel(dto, { connectionString: '' }),
    smtpSettings: toFormModel(dto.smtpSettings ?? new EmailSettingsEditDto(), { smtpHost: '' }),
});
```

For more details, see Angular's [Migrating existing forms to Signal Forms](https://angular.dev/guide/forms/signals/migration) guide.

## Rules Instead of Attributes

| Template-driven | Signal Forms schema |
|---|---|
| `required` | `required(product.name)` |
| `[required]="condition"` | `required(product.name, { when: () => condition })` |
| `minlength="2"`, `maxlength="128"` | `minLength(product.name, 2)`, `maxLength(product.name, 128)` |
| `min="0"`, `max="1000"` | `min(product.price, 0)`, `max(product.price, 1000)` |
| `pattern="^[A-Z]+$"` | `pattern(product.code, /^[A-Z]+$/)` |
| `type="email"` with an e-mail pattern | `email(product.contactEmail)` |
| `[disabled]="condition"` | `disabled(product.code, () => condition)` |
| A field that is shown only when a condition is true | `hidden(product.field, () => !condition)`, and remove the field from the page with `@if (!productForm.field().hidden())`. Hidden fields are not validated. |
| `validateEqual="password"` | `mustMatch(user.passwordRepeat, user.password)` |
| `[requireDigit]`, `[requiredLength]` and the other password attributes | `passwordComplexity(user.password, () => this.passwordComplexitySetting())` |
| A custom validator directive | `validate(product.field, ({ value }) => (isValid(value()) ? null : { kind: 'myError' }))` |
| `productForm.form.valid` | `productForm().valid()` |

Show the password and custom errors with the matching messages:

```html
<validation-messages [field]="userForm.password" [errorDefs]="passwordComplexityErrorDefs" />
<validation-messages
    [field]="userForm.passwordRepeat"
    [errorDefs]="passwordComplexityErrorDefs"
    [customMessages]="{ mustMatch: 'PasswordsDontMatch' }" />
```

To set a field from code, for example after a lookup, write its value directly: `this.productForm.price().value.set(result.price)`. Values set from code do not mark the field dirty.

## Controls

| Control | Binding and model value |
|---|---|
| `<input type="text">`, `<textarea>`, `<input type="password">` | `[formField]`, model value `''` (never `null`) |
| `<input type="number">` | `[formField]`, model value `number` or `null` |
| `<input type="checkbox">` | `[formField]`, model value `boolean` |
| `nz-select`, `nz-switch` and the other ng-zorro inputs | `[formField]` |
| `abp-datetime-picker`, `timezone-combo`, `role-combo`, `permission-combo`, inputs with `[imask]` | `[formField]` |
| `<select>` with string option values | `[formField]`, model value `''`. A nullable string model does not compile. |
| `<select>` with numeric or enum values, `<select multiple>` | Use `nz-select` with `[formField]`. A native select always writes a string, and Signal Forms does not support `<select multiple>`. |
| `nz-date-picker` with `datePickerLuxonModifier` | No `[formField]`: bind `[date]` and set the field in `(dateChange)` |
| Your own input component | Implement `FormValueControl<T>` with `readonly value = model<T>()`. See `password-input-with-show-button`. |

## Things to Watch

- `[formField]` sets the `name` attribute of an input to a generated value. UI tests must find fields by `id`, not by `name`.
- The compiler rejects attribute bindings such as `[attr.maxlength]`, `[attr.pattern]`, `[name]` and `[required]` on a `[formField]` element (NG8022).
- `required` does not reject an empty array. For a multi-select, use `validate(path, ({ value }) => (value().length ? null : { kind: 'required' }))`.
- Without `FormsModule`, a `<form>` element that has no `[formRoot]` is submitted by the browser and reloads the page when Enter is pressed. Add `(submit)="$event.preventDefault()"` to such a form.
- `XxxDto.fromJS(...)` parses `DateTime` values again and returns them in the local time zone. When a value comes from a UTC picker and the server expects it as picked, assign the original `DateTime` after `fromJS`.

## Power Tools

Power Tools chooses the form style by the version of your project:

- **v16.0 and later:** The create and edit components (modal, full page and offcanvas), the create or edit modal of a master-detail child entity and the lookup create modal use Signal Forms. Multi-lingual entities keep Reactive Forms with a form array of translations, without `ngModel`.
- **v15.5:** The generated code keeps template-driven forms. These templates are in `FileTemplates/Client/Angular/NgModelForms`.

Pages that you generated before the upgrade keep working. When you regenerate an entity, its create and edit components move to Signal Forms, and the code in your `.component.ts` subclasses and in the `az:begin` / `az:end` regions is preserved. Update that code for the new form:

- Markup in a `form-fields` region that uses `[(ngModel)]` must use `[formField]`, because the regenerated component no longer imports `FormsModule`. Add each custom field and its rules to the `.custom-fields.ts` file that Power Tools generates next to the component. See [Custom Code - Angular](Power-Tools-Custom-Code-Angular.md).
- Code that reads or writes the entity object, such as `this.product.name`, must read `this.product().name` and write `this.productForm.name().value.set(...)`.

If you customized the `CreateEditTemplate` or `LookupCreateModalTemplate` templates, your `.custom.txt` files were written for template-driven forms. Re-create them from the new default templates after the upgrade. See [How to Create & Edit Power Tools Templates](How-To-Create-Edit-Power-Tools-Templates.md).
