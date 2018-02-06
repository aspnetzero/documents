### Introduction

 In this document, we will introduce **ASP.NET Zero Power Tools** and explain it. This tool is developed to minimize the effort of creating a new entity. It creates all related layers (including UI) by defining an entity.

### Download And Install

 If your project version is 5.1.0+, all you have to do is just installing **ASP.NET Zero Power Tools** extension on visual studio, from https://marketplace.visualstudio.com/items?itemName=Volosoft.AspNetZeroPowerTools.

 If your project version is below 5.1.0, you also have to copy the AspNetZeroRadTool folder to your own project, from a newly downloaded 5.1.0+ project.
 
### How To Use It?
 
 Extension is inside **Tools** menu (Tools -> Asp.Net Zero -> Create An Entity). When you run it, you will see the interface for creating an entity. After carefully filling the fields, press **Generate** button to start the code generation process. 
 
 <img src="images/RadToolUI.jpg" alt="Extension UI" class="img-thumbnail" width="507" height="440" />
 
 A simple console will appear and give you information about the process. If there is no warning and fail, run your project to see the results (Angular developers will have to run NSwag manually). You probably won't see a new page, but don't worry and grant yourself the required **permissions**.
 
 Warning: Be sure that you have saved your works before running this tool, since it will add new files and modify some of the existing files. We strongly recommend using a source control system (like Git). Otherwise, backup your project.
 
### How It Works?
 
 DLLs, that is inside the folder mentioned above, do all the work. The extension contains just a user interface. This design is required, otherwise it would be available for only visual studio windows users. But since the tool is built on .NET Core platform, **Mac** or **Linux** users can safely use the tool. On these operating systems, you have to manually do the work that is done by the extension, which is just creating a short and basic JSON file as input.

### Generated Files
	
 Here is the full list of the files that are created or modified by the tool, if you give a basic "Cars" entity as input.

#### Server Side

###### Created:

 -   Car.cs
 -   CarDto.cs
 -   CreateOrEditCarDto.cs
 -   DeleteCarInput.cs
 -   GetCarInput.cs
 -   GetAllCarsInput.cs
 -   CarConsts.cs
 -   ICarAppService
 -   CarAppService
 
###### Modified:

 -   AppAuthorizationProvider.cs
 -   AppPermissions.cs
 -   ProjectNameDbContext.cs
 -   CustomDtoMapper.cs
 -   ProjectName.xml (English localization file)
 
 (also adds a database migration and updates the database, optionally.)

#### Client Side

##### Angular

###### Created:

 -   cars.component.ts
 -   cars.component.html
 -   create-or-edit-car-modal.component.ts
 -   create-or-edit-car-modal.component.html

###### Modified:

 -   app-navigation.service.ts
 -   service-proxy.module.ts
 -   (Main or Admin)-routing.module.ts
 -   (Main or Admin).module.ts

##### Mvc

###### Created:

 -   CarsController.cs
 -   CarsViewModel.cs
 -   CreateOrEditCarViewModel.cs
 -   Index.js
 -   Index.cshtml
 -   createOrEditModal.js
 -   createOrEditModal.cshtml

###### Modified:

 -   (AppArea)NavigationProvider.cs
 -   (AppArea)PageNames.cs
 
 
 <img src="images/RadToolCarsTable.jpg" alt="Generated User Interface" class="img-thumbnail" width="1371" height="445" />