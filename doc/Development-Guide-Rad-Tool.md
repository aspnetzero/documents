
  <img src="images/RadToolCarsTable.jpg" alt="Generated User Interface" class="img-thumbnail" width="1371" height="445" />
  
### Introduction

 In this document, we will introduce **ASP.NET Zero Power Tools** and explain it. This tool is developed to minimize the effort of creating a new entity. It creates all related layers (including UI) by defining an entity.

### Download And Install

 If your project version is 5.1.0+, all you have to do is just installing **ASP.NET Zero Power Tools** extension on visual studio, from https://marketplace.visualstudio.com/items?itemName=Volosoft.AspNetZeroPowerTools.

 If your project version is below 5.1.0, you also have to copy the AspNetZeroRadTool folder to your own project, from a newly downloaded 5.1.0+ project.

 Tool doesn't support versions before v5.0.0.
 
### How To Use It?

 Extension is inside **Tools** menu (Tools -> Asp.Net Zero -> Create An Entity). When you run it, you will see the interface for creating an entity. After carefully filling the fields, press **Generate** button to start the code generation process. 

 <img src="images/RadToolUI.jpg" alt="Extension UI" class="img-thumbnail" width="507" height="440" />

 A simple console will appear and give you information about the process. If there is no warning and fail, run your project to see the results. You probably won't see a new page, but don't worry and grant yourself the required **permissions**.

Warning: If you are working on ASP.NET Core & Angular template, after generating the entity via Power Tools, run your ***.Web.Host** project and then run "**./angular/nswag/refresh.bat**" to update **service-proxies.ts**.

 Warning: Be sure that you have saved your works before running this tool, since it will add new files and modify some of the existing files. We strongly recommend using a source control system (like Git). Otherwise, backup your project.

### How It Works?

DLLs, that is inside the folder mentioned above, do all the work. The extension contains just a user interface. This design is required, otherwise it would be available for only visual studio windows users. But since the tool is built on .NET Core platform, **Mac** or **Linux** users can safely use the tool. On these operating systems, you have to manually do the work that is done by the extension, which is just creating a short and basic[ JSON file](https://aspnetzero.com/Documents/Development-Guide-Rad-Tool-Mac-Linux) as input.

### How To Edit Pre-defined Templates Or Create A New Template?

 The templates are inside "/AspNetZeroRadTool/FileTemplates" directory. Every template is splitted into 3 file ("MainTemplate.txt", "PartialTemplates.txt","TemplateInfo.txt") . If you want to edit any file, just replicate it in same directory and change it's extension to ".custom.txt" from ".txt".  For example, you can create "MainTemplate.custom.txt" to override "MainTemplate.txt" in same directory. Plase don't make any change on orginal templates.
 To create a new template, do it the way we do in pre-defined templates. Tool doesn't know any info about templates and explore them in "/FileTemplates" directory every time it is run. So your new template will be precessed like pre-defined ones. (".custom" extension is not needed for new templates.)

 You can request help from our support team on [github](github.com/aspnetzero/aspnet-zero-core)  or  [forum](https://forum.aspnetboilerplate.com/viewforum.php?f=5)  if you are struggling


### Generated Files

 Here is the full list of the files that are created or modified by the tool, if you give a basic "Cars" entity as input.

#### Server Side

**Created**

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

**Modified**

 -   AppAuthorizationProvider.cs
 -   AppPermissions.cs
 -   ProjectNameDbContext.cs
 -   CustomDtoMapper.cs
 -   ProjectName.xml (English localization file)

 (Also adds a database migration and updates the database, optionally.)


#### Client Side

##### Angular

**Created**

 -   cars.component.ts
 -   cars.component.html
 -   create-or-edit-car-modal.component.ts
 -   create-or-edit-car-modal.component.html
 -   Lookup-Table-modal.component.ts
 -   Lookup-Table-modal.component.html
 -   Lookup-Table-modal.component.less

**Modified**

 -   app-navigation.service.ts
 -   service-proxy.module.ts
 -   (Main or Admin)-routing.module.ts
 -   (Main or Admin).module.ts

##### Mvc

**Created**

 -   CarsController.cs
 -   CarsViewModel.cs
 -   CreateOrEditCarViewModel.cs
 -   Index.js
 -   Index.cshtml
 -   createOrEditModal.js
 -   createOrEditModal.cshtml
 -   LookupTableViewModel.cshtml
 -   LookupTableModal.js
 -   LookupTableModal.cshtml

**Modified**

 -   (AppArea)NavigationProvider.cs
 -   (AppArea)PageNames.cs
 
  (Lookup files are created per foreign key.)
 

 <img src="images/RadToolCarsTable.jpg" alt="Generated User Interface" class="img-thumbnail" width="1371" height="445" />