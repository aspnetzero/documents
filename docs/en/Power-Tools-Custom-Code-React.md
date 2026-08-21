# Power Tools Custom Code - React

ASP.NET Zero Power Tools generates code that can be fully regenerated whenever you change an entity. The **Custom Code** feature lets you add your own code to the generated pages and classes so that your additions survive regeneration.

This page covers how Custom Code works in **React** projects. For other UI frameworks.

## Enabling Custom Code

Open Power Tools, select your entity, and turn on the **Generate Overridable Entity** switch in the Entity Information section.

<img src="images/aspnet-zero-power-tools-generate-overridable-entity.png">

When this switch is on, Power Tools generates additional customization files that you own. These files are created once and never overwritten, even when you regenerate the entity.

> The switch affects both the server side and the client side.

## How It Works

React uses a different approach than Angular and MVC. Instead of inline region markers or base/subclass patterns for UI files, React generates **separate customization files** with typed hook functions that the generated page imports and calls at fixed extension points.

| Mechanism | How It Works | Applies To |
|---|---|---|
| **Extension files** (SkipIfExists) | A base class is regenerated every time. A developer-owned subclass is generated once and never overwritten. | Server-side C# files only |
| **Customization files** | Separate `.tsx` files with hook functions. Never overwritten. The generated page imports and calls them. | React pages, forms, and detail views |
| **Regenerated type definitions** | A `customizations.types.ts` file that is regenerated to keep TypeScript types in sync with the entity. | React customization contexts |

## Server Side

When **Generate Overridable Entity** is enabled, the following server-side extension files are generated. These are shared across all UI frameworks.

| File | Base Class | Purpose |
|---|---|---|
| `AppService.Extended.cs` | `AppServiceBase` | Override application service methods |
| `AppServiceInterface.Extended.cs` | - | Extend the service interface |
| `Entity.Extended.cs` | `EntityBase` | Add custom properties or methods to the entity |
| `CreateOrEditDto.Extended.cs` | `CreateOrEditDtoBase` | Extend the input DTO |
| `EntityDto.Extended.cs` | `EntityDtoBase` | Extend the list DTO |
| `GetAllInput.Extended.cs` | `GetAllInputBase` | Add custom filter parameters |
| `GetAllForExcelInput.Extended.cs` | `GetAllForExcelInputBase` | Add custom Excel export filter parameters |
| `GetAllOutput.Extended.cs` | `GetAllOutputBase` | Extend the output DTO |
| `GetForEditOutput.Extended.cs` | `GetForEditOutputBase` | Extend the edit output DTO |
| `LookupDto.Extended.cs` | `LookupDtoBase` | Extend the lookup DTO |
| `HostController.Extended.cs` | `HostControllerBase` | Extend the API controller |

Each `.Extended.cs` file contains:

```csharp
// Write your custom code here.
// ASP.NET Zero Power Tools will not overwrite this class
// when you regenerate the related entity.
```

The base files (without `.Extended`) are fully regenerated on every run. New entity properties automatically flow into them. Your customizations in the extension files are preserved because Power Tools never touches them after the first generation.

## React Customization Files

When you enable **Generate Overridable Entity**, Power Tools generates the following files alongside the main page:

| Generated File | Overwritten? | Purpose |
|---|---|---|
| `customizations.tsx` | **Never** | Hook functions for the list page: extra columns, row actions, toolbar actions |
| `customizations.types.ts` | **Yes** (regenerated) | TypeScript types for the customization context |
| `CreateOrEdit*Modal.custom.tsx` | **Never** | Hook functions for the form: extra fields, before/after save |
| `View*Modal.custom.tsx` | **Never** | Hook functions for the detail view: extra detail items |

> `customizations.types.ts` is the only customization-related file that gets regenerated. This is intentional, when the entity shape changes, the type definitions update and any stale code in your customization files produces a compile error rather than breaking silently.

### File Layout Example for a `Product` Entity

```
pages/admin/inventory/products/
├── index.tsx                                    ← regenerated (main page)
├── customizations.tsx                           ← yours (list page hooks)
├── customizations.types.ts                      ← regenerated (types)
├── components/
│   ├── CreateOrEditProductModal.tsx              ← regenerated
│   ├── CreateOrEditProductModal.custom.tsx       ← yours (form hooks)
│   ├── ViewProductModal.tsx                      ← regenerated
│   └── ViewProductModal.custom.tsx              ← yours (detail hooks)
```

## List Page Hooks

The `customizations.tsx` file provides three hook functions for the list page:

```tsx
import type { ReactNode } from "react";
import type {
  ProductColumn,
  ProductRowAction,
  ProductsPageContext,
} from "./customizations.types";

/** Columns appended after the generated ones. */
export const extraColumns = (ctx: ProductsPageContext): ProductColumn[] => [];

/** Extra entries for a row's action menu, appended after the generated ones. */
export const extraRowActions = (
  ctx: ProductsPageContext,
  record: Parameters<NonNullable<ProductColumn["render"]>>[1],
): ProductRowAction[] => [];

/** Extra buttons in the page header, rendered after the generated ones. */
export const extraToolbarActions = (ctx: ProductsPageContext): ReactNode => null;
```

The generated `index.tsx` imports these functions and calls them at the right points:

- `extraColumns` is spread into the Ant Design table column definitions
- `extraRowActions` is pushed into each row's action dropdown menu
- `extraToolbarActions` is rendered in the page header area

### Example - Adding a Custom Column

```tsx
export const extraColumns = (ctx: ProductsPageContext): ProductColumn[] => [
  {
    title: "Custom Score",
    dataIndex: ["product", "customScore"],
    render: (text: any) => <span>{text ?? "-"}</span>,
  },
];
```

### Example - Adding a Custom Toolbar Button

```tsx
export const extraToolbarActions = (ctx: ProductsPageContext): ReactNode => (
  <button
    className="btn btn-outline-primary btn-sm"
    onClick={() => ctx.navigate("/app/admin/reports/products")}
  >
    Product Report
  </button>
);
```

### Example - Adding a Custom Row Action

```tsx
export const extraRowActions = (
  ctx: ProductsPageContext,
  record: Parameters<NonNullable<ProductColumn["render"]>>[1],
): ProductRowAction[] => [
  {
    key: "duplicate",
    label: "Duplicate",
    onClick: () => {
      // your duplication logic here
    },
  },
];
```

## Form Hooks

The `CreateOrEdit*Modal.custom.tsx` file provides three hook functions for the create/edit form:

```tsx
import type { ReactNode } from "react";
import type { CreateOrEditProductFormContext } from "../customizations.types";
import type { CreateOrEditProductDto } from "@api/generated/service-proxies";

/** Form items rendered after the generated ones. */
export const extraFormFields = (
  ctx: CreateOrEditProductFormContext,
): ReactNode => null;

/**
 * Runs after validation, just before the record is sent to the server.
 * Adjust `dto` in place to add computed values.
 * Throw to cancel the save.
 */
export const beforeSave = async (
  ctx: CreateOrEditProductFormContext,
  dto: CreateOrEditProductDto,
): Promise<void> => {};

/** Runs after the record is saved and before the modal closes. */
export const afterSave = async (
  ctx: CreateOrEditProductFormContext,
  dto: CreateOrEditProductDto,
): Promise<void> => {};
```

The generated `CreateOrEditProductModal.tsx` calls these hooks:

- `extraFormFields` is rendered after the generated form fields inside the `<Form>`
- `beforeSave` is called after form validation, before the API call -- **throw to cancel the save**
- `afterSave` is called after a successful save, before the modal closes

### Example - Adding a Custom Form Field

```tsx
export const extraFormFields = (
  ctx: CreateOrEditProductFormContext,
): ReactNode => (
  <Form.Item
    label="Internal Notes"
    name="internalNotes"
    rules={[{ max: 500, message: "Maximum 500 characters" }]}
  >
    <Input.TextArea rows={3} />
  </Form.Item>
);
```

### Example - Modifying DTO Before Save

```tsx
export const beforeSave = async (
  ctx: CreateOrEditProductFormContext,
  dto: CreateOrEditProductDto,
): Promise<void> => {
  // Set a computed field before sending to the server
  dto.slug = dto.name?.toLowerCase().replace(/\s+/g, "-");
};
```

## Detail View Hooks

The `View*Modal.custom.tsx` file provides one hook function for the detail/view modal:

```tsx
import type { ReactNode } from "react";
import type { ViewProductContext } from "../customizations.types";

/** Detail items rendered after the generated ones. */
export const extraDetailItems = (ctx: ViewProductContext): ReactNode => null;
```

### Example - Adding Custom Detail Fields

```tsx
export const extraDetailItems = (ctx: ViewProductContext): ReactNode => (
  <>
    <Descriptions.Item label="Calculated Margin">
      {ctx.entity?.margin ? `${ctx.entity.margin}%` : "-"}
    </Descriptions.Item>
  </>
);
```

## Context Objects

Each hook receives a typed context object. The context provides access to useful values without coupling your code to the generated component's internals.

### Page Context

| Property | Type | Description |
|---|---|---|
| `refresh` | `() => void` | Reloads the table data |
| `isGranted` | `(permission: string) => boolean` | Checks a permission |
| `navigate` | React Router `navigate` | Navigates to a route |

### Form Context

| Property | Type | Description |
|---|---|---|
| `form` | Ant Design `FormInstance` | The form instance for reading/setting field values |
| `isEditMode` | `boolean` | Whether the form is editing an existing record |
| `entityId` | Primary key type or `undefined` | The entity's primary key (in edit mode) |
| `service` | Service proxy instance | The service proxy for making API calls |

## Safety Features

- **SkipIfExists:** Customization files (`customizations.tsx`, `*.custom.tsx`) are generated once and never overwritten, even on regeneration.
- **Type safety:** `customizations.types.ts` is regenerated to keep types in sync. If the entity changes shape, stale code in your customization files produces a compile error rather than breaking silently.
- **Ownership guard:** When you toggle the **Generate Overridable Entity** switch on an already-generated entity, Power Tools detects whether existing files contain developer code and refuses to overwrite them.

## Next

[How to Create & Edit Power Tools Templates](How-To-Create-Edit-Power-Tools-Templates.md)
