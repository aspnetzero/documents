# How to Create & Edit Power Tools Templates

Power Tools uses text templates for code generation. Templates are stored in the project's `AspNetZeroRadTool/FileTemplates` directory.

Default templates are bundled with the global tool and synchronized into the project when Power Tools opens a project or starts generation. User customizations are stored as `.custom.txt` files and are preserved when default templates are refreshed.

For day-to-day customization, use the **Templates** page in the Power Tools web UI. You can select an existing template, customize it in the browser, compare it with the default template, and insert placeholders without manually creating `.custom.txt` files.

## Template Files

Each template folder can contain these files:

**MainTemplate.txt:** Main code generation template.

**PartialTemplates.txt:** Conditional and looped template fragments used by `MainTemplate.txt`.

**TemplateInfo.txt:** Output path and generation condition for the template.

*Folder structure*

![Folder structure](images/power-tools-folder-structure.png)

## Edit Pre-defined Templates

Do not edit the default `.txt` files directly. They can be refreshed by newer Power Tools versions.

Open the Power Tools web UI and go to the **Templates** page. The template tree shows server templates, the client templates for the current project type, test templates, and MAUI templates when they are available.

Select the template you want to change, then choose one of the editor tabs: **MainTemplate**, **PartialTemplates**, or **TemplateInfo**. Default templates are read-only. Click **Customize this template** to make the selected tab editable.

After editing, click **Save**. Power Tools writes your change to the matching `.custom.txt` file, such as `MainTemplate.custom.txt`, `PartialTemplates.custom.txt`, or `TemplateInfo.custom.txt`, and keeps the original `.txt` file unchanged.

Use **Diff** to compare your customization with the default template. For **MainTemplate**, you can also use **Code**, **Split**, and **Preview** modes. Preview uses sample placeholder values for visual review; it is not a replacement for running generation.

Use the placeholders panel on the right side to search placeholders and insert them into the editor.

![Templates page](images/power-tools-template-page.png)

## Revert a Custom Template

To revert a customized template file, select its tab in the template editor and click **Revert**. Power Tools deletes the related `.custom.txt` file and starts using the default `.txt` file again.

You can also revert manually by deleting the related `.custom.txt` file from the template folder.

## Templates for Older Project Versions

The Angular templates directly under `Client/Angular` are used for the current project versions. Power Tools keeps the templates of older Angular project versions in sub folders and chooses them by the version of your project:

| Folder | Used for |
|---|---|
| `Client/Angular` | v15.5 and later. On v16.0 and later, the create/edit templates use Signal Forms. |
| `Client/Angular/NgModelForms` | The template-driven (`ngModel`) `CreateEditTemplate`, `CreateEditTemplate.FullPage`, `CreateEditTemplate.Offcanvas`, `MasterDetailChildPage/CreateEditTemplate` and `LookupCreateModalTemplate` of v15.5 projects. They replace the templates with the same names in `Client/Angular`. |
| `Client/Angular/PrimeNg` | v15.3 and v15.4 (PrimeNG based pages) |
| `Client/Angular/Legacy` | Versions before v15.3 (module based pages) |

Customize the template in the folder that your project version uses. A `.custom.txt` file in another folder has no effect.

On a v15.5 project, the **Templates** page lists the `NgModelForms` templates under their usual names, such as `CreateEditTemplate`, so your customizations are saved to the folder that generation uses. Customizations that you made before `NgModelForms` existed are moved there automatically when Power Tools opens the project, or when you change the project version in the settings. Power Tools does not overwrite a customization that already exists in `NgModelForms`.

Customizations are stored per file. When a new Power Tools version changes the default templates, a customized `MainTemplate.custom.txt` is combined with the new default `PartialTemplates.txt`. After you upgrade your project, review your customized templates and re-create them from the new defaults. This applies to the create/edit templates when you upgrade an Angular project to v16.0, because they move from `ngModel` to Signal Forms. See [Migrating to Signal Forms](Migrating-To-Signal-Forms-Angular.md).

## Create New Templates

Choose a folder under `FileTemplates` based on where the generated file belongs:

* `Server`
* `Client/Angular`
* `Client/Mvc`
* `Client/React`
* `Test`
* `Maui`

Then create a new template folder and add `MainTemplate.txt`, `PartialTemplates.txt`, and `TemplateInfo.txt`. New template folders are created in the file system, not from the Templates page.

You can use existing templates as a reference. For guidance on placeholders and template behavior, see [Understanding Power Tools Templates](Power-Tools-Understanding-Power-Tools-Templates.md).

Power Tools discovers templates in the `FileTemplates` directory each time generation runs. If the web UI is open while you add new template folders manually, refresh the page if you do not see them immediately.
