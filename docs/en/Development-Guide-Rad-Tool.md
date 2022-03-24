# Development Guide

In this document, we will introduce **ASP.NET Zero Power Tools** and explain it. ASP.NET Zero Power Tools minimizes the effort for creating CRUD pages. It generates all the layers from the database to the user interface by just defining an entity. 

> ASP.NET Zero Power Tools supports ASP.NET Zero v5.0.0 and above versions.

## How To Use It?

You can use ASP.NET Zero Power Tools in two ways: 

1. as a Visual Studio Extension 
2. Using its DLL files directly with a command line. 

Visual Studio Extension is available for Windows version of Visual Studio. Mac and Linux users can use DLL of ASP.NET Zero Power Tools to generate CRUD pages.

Warning: If you are working on ASP.NET Core & Angular template, after generating the entity via Power Tools, run your ***.Web.Host** project and then run "**./angular/nswag/refresh.bat**" to update **service-proxies.ts**.

Warning: Be sure that you have saved your work before running this tool since it will add new files and modify some of the existing files. We strongly recommend using a source control system (like Git).  Otherwise, backup your project.

## How It Works?

DLLs (that are inside the ```aspnet-core\AspNetZeroRadTool``` folder in your solution) do all the work. The extension is just a user interface. Since the tool is built with .NET Core, **Mac** or **Linux** users can safely use it without the user interface.

Using ASP.NET Zero Power Tools on Mac and Linux requires a bit more effort. You have to create a [ JSON file](https://aspnetzero.com/Documents/Development-Guide-Rad-Tool-Mac-Linux) as input for code generation manually. For using ASP.NET Zero Power Power Tools in Mac or Linux, please check [Development Guide(Mac/Linux) document](Development-Guide-Rad-Tool-Mac-Linux).

Visual Studio extension of ASP.NET Zero Power Tools also uses this DLL file for code generation.

## Edit Pre-defined Templates

Power Tools uses text templates for code generation, and these templates are located inside ```/AspNetZeroRadTool/FileTemplates``` directory in your project's root directory. Each template is split into three files:

* **MainTemplate.txt**: Power Tools uses this template for main code generation.  
* **PartialTemplates.txt**: Power Tools renders some placeholders in MainTemplate.txt conditionally. These conditional templates are stored in PartialTemplates.txt.
* **TemplateInfo.txt**: Stores information about the template like path and condition.

If you want to edit any file, copy it in the same directory and change it's an extension to ```.custom.txt``` from ```.txt```. For example, you can create ```MainTemplate.custom.txt``` to override ```MainTemplate.txt``` in the same directory. Please don't make any changes to the original templates.

## Create A New Template

You can also create new templates for code generation. Power Tools will generate new files based on your new templates during code generation. To create a new template, do the same process as editing a pre-defined template. 

Power Tools discovers templates in the ```/FileTemplates``` directory every time it is run. So, restarting Power Tools will find your newly created templates.

## Change Destination Path Of New Files

To change the destination path of a template, find the template folder of it in "AspNetZeroRadTool/FileTemplates" directory and edit the content of **TemplateInfo.txt** file.

Also, if you have moved a file that is going to be modified during rad tool generation, you can modify "AspNetZeroRadTool/**config.json**" file and set the new path of this file.

## Generated Files

Here is the full list of the files that are created or modified by the tool, if you give a basic "Cars" entity as input.

### Server Side

**Files that are being created**

 -   Car.cs
 -   CarDto.cs
 -   LookupDto.cs
 -   GetAllForLookupTableInput.cs
 -   GetCarForEditOutput.cs
 -   GetAllCarsOutput.cs
 -   CreateOrEditCarDto.cs
 -   GetAllCarsInput.cs
 -   CarConsts.cs
 -   ICarAppService
 -   CarAppService
 -   CarsExcelExporter.cs
 -   ICarsExcelExporter.cs

**Files that are being modified**

 -   AppAuthorizationProvider.cs
 -   AppPermissions.cs
 -   ProjectNameDbContext.cs
 -   CustomDtoMapper.cs
 -   ProjectName.xml (English localization file)

 (Optionally, adds a database migration and updates the database.)


### Client Side

#### Angular

**Files that are being created**

 -   cars.component.ts
 -   cars.component.html
 -   create-or-edit-car-modal.component.ts or create-or-edit-car.component.ts (if "Create Non-modal  CRUD Page" is selected)
 -   create-or-edit-car-modal.component.html or create-or-edit-car.component.html (if "Create Non-modal  CRUD Page" is selected)
 -   view-car-modal.component.ts or view-car.component.ts (if "Create Non-modal  CRUD Page" is selected)
 -   view-car-modal.component.html or view-car.component.ts (if "Create Non-modal  CRUD Page" is selected)
 -   Lookup-Table-modal.component.ts
 -   Lookup-Table-modal.component.html
 -   Lookup-Table-modal.component.less

**Files that are being modified**

 -   app-navigation.service.ts
 -   service-proxy.module.ts
 -   (Main or Admin)-routing.module.ts
 -   (Main or Admin).module.ts

#### Mvc

**Files that are being created**

 -   CarsController.cs
 -   CarsViewModel.cs
 -   CreateOrEditCarViewModel.cs
 -   Index.js
 -   Index.cshtml
 -   CreateOrEditModal.js or CreateOrEdit.js (if "Create Non-modal  CRUD Page" is selected)
 -   CreateOrEditModal.cshtml  or CreateOrEdit.cshtml (if "Create Non-modal  CRUD Page" is selected)
 -   ViewCarModal.cshtml  or ViewCar.cshtml (if "Create Non-modal  CRUD Page" is selected)
 -   LookupTableViewModel.cshtml
 -   LookupTableModal.js
 -   LookupTableModal.cshtml

**Files that are being modified**

 -   (AppArea)NavigationProvider.cs
 -   (AppArea)PageNames.cs

>   Note that lookup files are being created per foreign key.

## Final Result

The below image shows a generated page which list records, allows filtering, inserting, deleting, updating and exporting excel functionalities.![Final result: generated page](images/RadToolCarsTable3.jpg)

This is the record edit model where you can update an existing record.
![Edit model](images/RadToolEditModal.jpg)
