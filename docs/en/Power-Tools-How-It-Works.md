# ASP.NET Zero Power Tools - How It Works

ASP.NET Zero Power Tools is installed as a .NET global tool and launched with the `aspnetzero-powertools` command. In its default mode, it starts a local ASP.NET Core web application and serves the Power Tools web UI.

## Runtime Modes

Power Tools has two user-facing runtime modes:

1. **Web UI mode:** `aspnetzero-powertools [--project <path>] [--port <number>] [--no-launch]`
2. **Headless mode:** `aspnetzero-powertools <entity.json> [--verbose] [--format] [--no-format] [--no-wait]`

The web UI is the recommended mode. Headless mode is useful for automation or for users who already maintain entity JSON files manually.

## Project Detection

When Power Tools starts, it locates the active ASP.NET Zero project in this order:

1. The path passed with `--project`.
2. The last project selected in a previous Power Tools session.
3. The current directory or one of its parent directories.

Power Tools looks for an `AspNetZeroRadTool/config.json` file. If it finds a `.sln` or `.slnx` file but no `config.json`, the web UI opens the first-time setup flow.

The last selected project is stored on the local machine under the user's application data folder. Project-specific generation files still remain inside the ASP.NET Zero solution.

## Working Folder

Power Tools uses the solution's `AspNetZeroRadTool` folder as its project working folder. This folder contains:

* `config.json`, which stores project settings, file locations, license information, and generation workflow steps.
* Entity JSON files such as `Products.Product.json`.
* `FileTemplates`, which contains default templates and any `.custom.txt` template overrides.
* Runtime files copied by the global tool when a generation starts.

The global tool is the installed application. The `AspNetZeroRadTool` folder is the project workspace used by the generator.

## Template Synchronization

Default Power Tools templates are bundled with the global tool. When a project is opened or generation starts, Power Tools syncs the bundled templates into the project's `AspNetZeroRadTool/FileTemplates` folder.

Default `.txt` template files can be refreshed by newer tool versions. User-created `.custom.txt` files are preserved and are used instead of the default template files.

## Generation Flow

When you click **Generate** in the web UI, Power Tools:

1. Saves or updates the entity JSON file in the working folder.
2. Syncs templates and deploys the generator runtime files into the working folder.
3. Runs the generator for the selected entity JSON.
4. Generates or modifies server-side code, client-side code, tests, localization, permissions, migrations, and optional MAUI files based on the selected options.
5. Runs generated-file formatting when enabled.
6. Runs generation workflow commands configured in `config.json`.
7. Streams the output to the browser.

## Generation Workflow

The generation workflow is configured in **Project Settings**. It always includes the code generation step and can include extra command steps.

Default workflows can run tasks such as:

* Creating MVC bundles.
* Creating Angular dynamic bundles.
* Running the Web.Host project and refreshing Angular service proxies.
* Installing React dependencies and refreshing React service proxies.

Custom command steps run relative to the ASP.NET Zero project root. For long-running commands such as `dotnet run`, a success pattern can be configured so the workflow continues when the expected output appears.

## Example of Generated Files

Here is the full list of the files that are created or modified by the tool, if you give a basic **Cars** entity as input.

### Server Side

**Files that are being created**

* Car.cs
* CarDto.cs
* LookupDto.cs
* GetAllForLookupTableInput.cs
* GetCarForEditOutput.cs
* GetAllCarsOutput.cs
* CreateOrEditCarDto.cs
* GetAllCarsInput.cs
* CarConsts.cs
* ICarAppService
* CarAppService
* CarsExcelExporter.cs
* ICarsExcelExporter.cs

**Files that are being modified**

* AppAuthorizationProvider.cs
* AppPermissions.cs
* ProjectNameDbContext.cs
* ProjectName.xml (English localization file)

**Note:** Power Tools automatically generates Mapperly mapper classes for entity to DTO mappings. Unlike the previous AutoMapper approach that used a centralized `CustomDtoMapper.cs` file, Mapperly uses individual mapper classes that are created per entity.

(Optionally, adds a database migration and updates the database.)

### Client Side (Angular)

**Files that are being created**

* cars.component.ts
* cars.component.html
* create-or-edit-car-modal.component.ts or create-or-edit-car.component.ts (if a full-page create/edit template is selected)
* create-or-edit-car-modal.component.html or create-or-edit-car.component.html (if a full-page create/edit template is selected)
* view-car-modal.component.ts or view-car.component.ts (if a full-page detail template is selected)
* view-car-modal.component.html or view-car.component.html (if a full-page detail template is selected)
* Lookup-Table-modal.component.ts
* Lookup-Table-modal.component.html
* Lookup-Table-modal.component.less

**Files that are being modified**

* app-navigation.service.ts
* service-proxy.module.ts
* (Main or Admin)-routing.module.ts
* (Main or Admin).module.ts

### Client Side (jQuery & MVC)

**Files that are being created**

* CarsController.cs
* CarsViewModel.cs
* CreateOrEditCarViewModel.cs
* Index.js
* Index.cshtml
* CreateOrEditModal.js or CreateOrEdit.js (if a full-page create/edit template is selected)
* CreateOrEditModal.cshtml or CreateOrEdit.cshtml (if a full-page create/edit template is selected)
* ViewCarModal.cshtml or ViewCar.cshtml (if a full-page detail template is selected)
* LookupTableViewModel.cshtml
* LookupTableModal.js
* LookupTableModal.cshtml

**Files that are being modified**

* (AppArea)NavigationProvider.cs
* (AppArea)PageNames.cs

### Client Side (React)

**Files that are being created**

* pages/admin/{namespace}/{entities}/index.tsx
* pages/admin/{namespace}/{entities}/components/CreateOrEditEntityModal.tsx or a full-page create/edit component
* pages/admin/{namespace}/{entities}/components/ViewEntityModal.tsx or a full-page detail component
* pages/admin/{namespace}/{entities}/components/ForeignEntityLookupTableModal.tsx for navigation properties

**Files that are being modified**

* lib/navigation/appNavigation.tsx
* routes/AppRouter.tsx

### Tests

If you select options to create Unit Tests or UI Tests, Power Tools creates files for unit testing the generated application service and files for running Playwright UI tests for generated UI pages.

> Lookup files are created per foreign key.
