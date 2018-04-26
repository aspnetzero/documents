
### Introduction

 In this document, we will explain how to use **ASP.NET Zero Power Tools** without the Visual Studio extension.

 Purpose of the **ASP.NET Zero Power Tools VS Extension** is to create an input file for the rad tool. So, in order to use it without extension, input file should be created manually. 

### Sample Input File

 Input file includes a JSON string and it's name is the same as your entity's name. Here is a sample Product.json:

    {
      "MenuPosition": "main",
      "RelativeNamespace": "Products",
      "EntityName": "Product",
      "EntityNamePlural": "Products",
      "TableName": "Products",
      "PrimaryKeyType": "int",
      "BaseClass": "Entity",
      "AutoMigration": true,
      "UpdateDatabase": true,
      "CreateUserInterface": true,
      "CreateViewOnly": true,
      "PagePermission": {
           "Host": true,
           "Tenant": true
      },
      "Properties": [
                {
                  "Name": "Name",
                  "Type": "string",
                  "MaxLength": 25,
                  "MinLength": 2,
                  "Range": {
                    "IsRangeSet": false,
                    "MinimumValue": 0,
                    "MaximumValue": 0
                  },
                  "Required": true,
                  "Nullable": false,
                  "Regex": "",
                  "UserInterface": {
                    "List": true,
                    "AdvancedFilter": true,
                    "CreateOrUpdate": true
                     }
                },
                {
                  "Name": "Type",
                  "Type": "ProductType",
                  "MaxLength": 0,
                  "MinLength": 0,
                  "Range": {
                    "IsRangeSet": false,
                    "MinimumValue": 0,
                    "MaximumValue": 0
                  },
                  "Required": false,
                  "Nullable": false,
                  "Regex": "",
                  "UserInterface": {
                    "List": true,
                    "AdvancedFilter": true,
                    "CreateOrUpdate": true
                  }
                }
      ],
      "NavigationProperties": [
				{
                  "Namespace": "Volosoft.RadToolExplainer.Authorization.Users",
                  "ForeignEntityName": "User",
                  "IdType": "long",
                  "IsNullable": true,
                  "PropertyName": "UserId",
                  "DisplayPropertyName": "Name",
                  "DuplicationNumber": 0,
                  "RelationType": "single"
                }
	   ],
      "EnumDefinitions": [
                {
                  "Name": "ProductType",
                  "Namespace": "Volosoft.RadToolExplainer",
                  "EnumProperties": [
                            {
                              "Name": "Liquid",
                              "Value": 1
                            },
                            {
                              "Name": "Solid",
                              "Value": 2
                            }
                   ]
                }
       ]
    }

### Guide To Create A New Input File

You have to fill the fields of json file according to your entity. However, some of the fields must match our constants. Here is the list of them:

- MenuPosition : "main" | "admin"
  
- RelativeNamespace: Namespace of your new entity. Doesn't include your company and project name.
  
- PrimaryKeyType: "int" | "long" | "Guid"
  
- BaseClass: "Entity" | "AuditedEntity" | "CreationAuditedEntity" | "FullAuditedEntity"
     
- RelationType: Only "single" is valid for now.
     
- DuplicationNumber: A number post-fix that is added to end of variable names to prevent duplicate if there are more than one foreign key addressing to same entity. '0' means no post-fix.  


#### Properties

 Properties are written as an array in JSON file. Add an object to that array for every property of your entity. There will be some unnecessary fields depending on property type. For example, you don't have to set regular expression for a numeric property or don't have to set range for a string. 

A property should be one of those types:

 - bool
 - byte
 - short 
 - DateTime
 - decimal
 - double
 - Guid
 - int
 - long
 - string
 
 or one of the enums you declared in "EnumDefinitions".

### How To Run Rad Tool

You can use that command to run it on any device:

    dotnet AspNetZeroRadTool.dll YourEntity.Json

### Notes

 - AspNetZeroRadTool folder is placed in your solution's directory. You have to place the json file and run the command in there.
 - Please Keep in mind that JSON file is completely case sensitive. 
 - Auto add-migration and update-database functions are disabled.
 - If you are working on ASP.NET Core & Angular template, after generating the entity via Power Tools, run your ***.Web.Host** project and then run "**./angular/nswag/refresh.bat**" to update **service-proxies.ts**.