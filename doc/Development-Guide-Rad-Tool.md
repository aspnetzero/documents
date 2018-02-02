### Introduction

 In this document, we will introduce **ASP.NET Zero Power Tool** and expalin it.

### Purpose Of The Tool

This tool is developed to minimize the effort of creating a new table. It creates all related layers (including UI) by defining an entity.

### Download And Install

 If your project version is 5.1.0+, all you have to do is just installing **ASP.NET Zero Power Tools** extension on visual studio, from https://marketplace.visualstudio.com/items?itemName=Volosoft.AspNetZeroPowerTools.

 If your project version is below 5.1.0, you also have to copy the AspNetZeroRadTool folder to your own project, from a newly downloaded 5.1.0+ project.
 
### How To Use It?
 
 Extension is inside **Tools** menu (tools-> Asp.Net Zero -> Create An Entity). When you run it, you will see the interface for creating an entity. After carefully filling the fields, press **Generate** button to start the code generation process. 
 
 A simple console will appear and give you information about the process. If there is no warning and fail, run your project to see the results (Angular developers will have to run nswag manually). You probably won't see a new page, but don't worry and give yourself a **permission**.
 
###  How It Works?
 
 Dll's, that is inside the folder mentioned above, do all the work. The extension contains just a user interface. This design is required, otherwise it would be available for only visual studio windows users. But since the tool is built on .Net Core platform, **mac** or **linux** users can safely use the tool. On these operation systems you have to manually do the work that is done by the extension, which is just creating a short and basic json file as input.