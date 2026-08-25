# Power Tools Custom Code - MAUI

ASP.NET Zero Power Tools generates code that can be fully regenerated whenever you change an entity. The **Custom Code** feature lets you add your own code to the generated pages and classes so that your additions survive regeneration.

This page covers how Custom Code works in the **MAUI Blazor Hybrid** mobile app. The MAUI app ships alongside every UI framework, so this page applies whether your web project is [Angular](Power-Tools-Custom-Code-Angular.md), [ASP.NET Core MVC](Power-Tools-Custom-Code-Mvc.md), or [React](Power-Tools-Custom-Code-React.md).

## Enabling Custom Code

Open Power Tools, select your entity, and turn on both the **Mobile** and **Generate Overridable Entity** switches in the Entity Information section.

When both switches are on, Power Tools generates additional customization files that you own. These files are created once and never overwritten, even when you regenerate the entity.

### Requirements

**Project version.** The MAUI customization files require a **v15.5+** project. MAUI code generation itself is supported from v13.3, so on a v13.3-v15.4 project the **Mobile** switch still produces pages, but the `.custom.cs` files and the `az:begin` slots are not part of them.

**Both switches.** **Mobile** alone generates the pages without customization files. **Generate Overridable Entity** alone customizes the server side and the web client but leaves the MAUI app untouched. You need both.

**Detail page.** `ViewProduct.custom.cs` is generated only when the entity's MAUI view page is enabled - the **Create View** option that appears under **Mobile** in the Visual Studio extension.

## How It Works

MAUI uses a third approach, different from Angular's base/subclass split and React's hook files:

| Mechanism | How It Works | Applies To |
|---|---|---|
| **Partial class files** (SkipIfExists) | A second `partial` of the same component class. Generated once, never overwritten. No base class and no wiring. | MAUI Razor component code (`.custom.cs`) |
| **Preserved region slots** (`az:begin` / `az:end`) | Named regions inside regenerated `.razor` markup. Power Tools preserves any markup you place between the markers. | MAUI Razor pages (`.razor`) |

Because your code is a partial of the *same* class rather than a subclass, there is no separate type to register and no route to update - the component is still `ProductIndex`. The generated `.razor.cs` beside it keeps being rewritten on every run, so new entity properties keep flowing in.

## Partial Class Files

For a `Product` entity, Power Tools generates these files under `<YourProject>.Maui\Pages\<Namespace>\`:

| Your file (never overwritten) | Regenerated file | Purpose |
|---|---|---|
| `ProductIndex.custom.cs` | `ProductIndex.razor.cs` | Add fields, methods, or overrides for the list page |
| `CreateOrEditProduct.custom.cs` | `CreateOrEditProduct.razor.cs` | Add fields, methods, or overrides for the create/edit page |
| `ViewProduct.custom.cs` | `ViewProduct.razor.cs` | Add fields, methods, or overrides for the detail page |

Each `.custom.cs` file starts out as an empty partial:

```csharp
namespace MyCompany.MyProject.Maui.Pages.Inventory;

public partial class ProductIndex
{
}
```

Add fields, methods, or overrides of virtual members from the component base class here. Because it is the same class, you can reach any `protected` member the generated `.razor.cs` declares without any extra plumbing.

### Example - Running Custom Logic After the Page Loads

```csharp
namespace MyCompany.MyProject.Maui.Pages.Inventory;

public partial class ProductIndex
{
    private int _lowStockCount;

    protected override async Task OnInitializedAsync()
    {
        await base.OnInitializedAsync();
        _lowStockCount = await CountLowStockAsync();
    }

    private Task<int> CountLowStockAsync()
    {
        // your logic here
        return Task.FromResult(0);
    }
}
```

## Razor - Preserved Region Slots

MAUI `.razor` pages contain named `az:begin` / `az:end` markers. You can place custom markup between these markers and it will be preserved on regeneration. The rest of the page is fully regenerated.

### Slot Reference

| Slot Name | File | Purpose |
|---|---|---|
| `list-fields` | `ProductIndex.razor` | Custom fields in each list row |
| `form-fields` | `CreateOrEditProduct.razor` | Custom form fields |
| `detail-fields` | `ViewProduct.razor` | Custom detail display fields |

### Example - Adding a Custom List Field

```html
<!-- az:begin(list-fields) -->
<div class="item-row">
    <span class="label">Margin</span>
    <span class="value">@(product.Margin?.ToString("P0") ?? "-")</span>
</div>
<!-- az:end(list-fields) -->
```

### Example - Adding a Custom Form Field

```html
<!-- az:begin(form-fields) -->
<div class="form-group">
    <label>Internal Notes</label>
    <InputTextArea @bind-Value="Model.InternalNotes" class="form-control" rows="3" />
</div>
<!-- az:end(form-fields) -->
```

After regeneration, your markup stays in place while the rest of the page is updated with any new entity properties.

## Preserved Region Rules

The `az:begin` / `az:end` markers follow these rules:

| Rule | Description |
|---|---|
| **Comment wrapper** | Must use HTML comment syntax in `.razor` files: `<!-- az:begin(name) -->` |
| **Region names** | Letters, digits, underscores, hyphens, and dots only |
| **No nesting** | Regions cannot be placed inside other regions |
| **No duplicates** | Each region name must be unique within a file |
| **Empty regions are no-ops** | If you don't add anything between the markers, the region is ignored |

## Server Side

The MAUI app consumes the same application services as the web client, so the server-side extension files are the ones described on the [Angular](Power-Tools-Custom-Code-Angular.md#server-side), [MVC](Power-Tools-Custom-Code-Mvc.md#server-side), and [React](Power-Tools-Custom-Code-React.md#server-side) pages. They are generated once for the entity regardless of which clients you target.

## Safety Features

Power Tools includes safety mechanisms to prevent code loss:

- **Abort on malformed markers:** If region markers are broken (unclosed, nested, duplicated, mismatched), the entire file is left untouched. No partial merge ever occurs.
- **Orphan rescue:** If a region name existed in the old file but not in the newly generated output (for example, if a template update removes a slot), the content is saved to a sidecar file (`<filename>.orphaned.txt`) and a warning is shown. Content is never silently dropped.
- **Ownership guard:** Toggling the **Generate Overridable Entity** switch on an entity that already has
  generated files is handled in both directions:
  - Turning it **off** points a fully generated template back at the file that holds your code. Power Tools
    detects your code and refuses to overwrite it.
  - Turning it **on** makes a path that used to be fully generated developer-owned. The file already sitting
    there is old generated code, so Power Tools **deletes it** and lets the template write the new split -
    saving a copy as `<filename>.bak` beside it first. If you had hand-edited that file, your changes are in
    the `.bak`, not in the regenerated one.

## Next

[How to Create & Edit Power Tools Templates](How-To-Create-Edit-Power-Tools-Templates.md)
