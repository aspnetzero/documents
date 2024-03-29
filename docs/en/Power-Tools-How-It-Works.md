# ASP.NET ZERO Power Tools - How it Works

DLLs (that are inside the `aspnet-core\AspNetZeroRadTool` folder in your solution) do all the work. The visual studio extension is just a user interface. Since the tool is built with .NET Core, Mac or Linux users can safely use it without the user interface.

ASP.NET Zero Power Tools uses JSON files as input. These JSON files are generated by the visual studio extension or manually. You can find the generated JSON files in the `aspnet-core\AspNetZeroRadTool` folder in your solution.

The tool generates code based on the json files. It uses the `aspnet-core\AspNetZeroRadTool\Templates` folder as template. You can create your own templates or modify the existing ones.

To create a custom template or edit an existing template, you can refer to the documentation provided at [How to Create & Edit Power Tools Templates](how-to-create-edit-power-tools-templates.md).

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
* CustomDtoMapper.cs
* ProjectName.xml (English localization file)

(Optionally, adds a database migration and updates the database.)

### Client Side (Angular)

**Files that are being created**

* cars.component.ts
* cars.component.html
* create-or-edit-car-modal.component.ts or create-or-edit-car.component.ts (if "Create Non-modal  CRUD Page" is selected)
* create-or-edit-car-modal.component.html or create-or-edit-car.component.html (if "Create Non-modal  CRUD Page" is selected)
* view-car-modal.component.ts or view-car.component.ts (if "Create Non-modal  CRUD Page" is selected)
* view-car-modal.component.html or view-car.component.ts (if "Create Non-modal  CRUD Page" is selected)
* Lookup-Table-modal.component.ts
* Lookup-Table-modal.component.html
* Lookup-Table-modal.component.less

**Files that are being modified**

* app-navigation.service.ts
* service-proxy.module.ts
* (Main or Admin)-routing.module.ts
* (Main or Admin).module.ts

### Client Side (jQuery & Mvc)

**Files that are being created**

* CarsController.cs
* CarsViewModel.cs
* CreateOrEditCarViewModel.cs
* Index.js
* Index.cshtml
* CreateOrEditModal.js or CreateOrEdit.js (if "Create Non-modal  CRUD Page" is selected)
* CreateOrEditModal.cshtml  or CreateOrEdit.cshtml (if "Create Non-modal  CRUD Page" is selected)
* ViewCarModal.cshtml  or ViewCar.cshtml (if "Create Non-modal  CRUD Page" is selected)
* LookupTableViewModel.cshtml
* LookupTableModal.js
* LookupTableModal.cshtml

### Tests

If you select options to create Unit Tests or UI Test on Power Tools, it will create files for unit testing the generated application service and files for running Playwright UI tests for generated UI pages.

**Files that are being modified**

* (AppArea)NavigationProvider.cs
* (AppArea)PageNames.cs

> Note that lookup files are being created per foreign key.