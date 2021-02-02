# Development Guide

## Introduction

In this document, we will explain how to use **ASP.NET Zero Power Tools** without the Visual Studio extension.

Purpose of the **ASP.NET Zero Power Tools VS Extension** is to create an input file. So, in order to use it without extension, input file needs to be created manually. 

You can find specification of the JSON file fields in the table below;

***Settings***

| Name                 | Description                                                  |
| :------------------- | ------------------------------------------------------------ |
| IsRegenerate         | Set `true` if you have generated this entity before.         |
| MenuPosition         | `main` or `administration`                                   |
| RelativeNamespace    | Namespace of your entity (not including project's namespace) |
| EntityName           | Entity Name                                                  |
| EntityNamePlural     | Entity Name Plural                                           |
| TableName            | Database Table Name (might be same with plural name)         |
| PrimaryKeyType       | Type of primay key. <br />Can be `int`, `long`, `string`, `Guid` |
| BaseClass            | Base class of your entity. <br />Can be `Entity`, `AuditedEntity`, `CreationAuditedEntity`, `FullAuditedEntity` |
| EntityHistory        | Set `true` to track history of this entity.                  |
| AutoMigration        | `true` add-migration automatically, `false` do not add migration (you need to add migration manually) |
| UpdateDatabase       | `true` update-database automatically, `false` do not update-database (you need to update-database manually) |
| CreateUserInterface  | `true` creates/modifies ui layer files                       |
| CreateViewOnly       | `true` creates a view-only modal in actions button in table of your entity in ui |
| CreateExcelExport    | `true` adds excel report button in ui                        |
| IsNonModalCRUDPage   | `true` creates non-modal pages.                              |
| PagePermission       | Multitenancy<br />`"PagePermission":{"Host": [ISHOSTALLOWED],"Tenant":[ISTENANTALLOWED]}` |
| Properties           | Properties of your entity. See 'Table 2' for more.           |
| NavigationProperties | Navigation properties of your entity. See 'Table 3' for more. |
| EnumDefinitions      | Enum definitions you use on your entity. See 'Table 4' for more. |



***Properties (Array):***

| Name          | Description                                                  |
| ------------- | ------------------------------------------------------------ |
| Name          | Property Name                                                |
| Type          | Type of property. <br />Can be `string`, `bool`, `byte`, `short`, `DateTime`, `decimal`, `double`, `Guid`, `int`, `long`, `enum` |
| MaxLength     | If type is `string` max length of string                     |
| MinLength     | If type is `string` min length of string                     |
| Range         | If type can have range value range of property<br />`"Range": {"IsRangeSet": [ISRANGESET],"MinimumValue": [MINVAL],"MaximumValue": [MAXVAL]}` |
| Required      | Is property required                                         |
| Nullable      | Is property nullable                                         |
| Regex         | specifies the regex that this property should match          |
| UserInterface | Will this property be listed, have a filter and editable in ui?<br />`"UserInterface": {"List": true,"AdvancedFilter": true,"CreateOrUpdate": true}` |



***NavigationProperties (Array):***

| Name                | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| Namespace           | Namespace of entity                                          |
| ForeignEntityName   | Entity Name                                                  |
| IdType              | Type of Foreign Key. See Entity -> PrimaryKeyType            |
| IsNullable          | Is nullable                                                  |
| PropertyName        | Property name (Property name for that entity which will store Foreign Key) |
| DisplayPropertyName | Property name of foreign entity. It will be displayed by that property on pages. |
| DuplicationNumber   | If you have two navigation property that navigates to same foreign entity, number them starting from 1. If not, just skip this. |
| RelationType        | `single` is the only option. |



***EnumDefinitions***

| Name           | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| Name           | Name                                                         |
| Namespace      | Namespace                                                    |
| EnumProperties | Properties <br />`"EnumProperties":[{"Name":"[PROPERYNAME]","Value":[PROPERYVALUE]}]` |



## Sample Input File

 Input file includes a JSON string and it's name is the same as your entity's name. Here is a sample Products.Product.json:

```json
{
  "IsRegenerate": false,
  "MenuPosition": "main",
  "RelativeNamespace": "Products",
  "EntityName": "Product",
  "EntityNamePlural": "Products",
  "TableName": "Products",
  "PrimaryKeyType": "int",
  "BaseClass": "Entity",
  "EntityHistory": false,
  "AutoMigration": true,
  "UpdateDatabase": true,
  "CreateUserInterface": true,
  "CreateViewOnly": true,
  "CreateExcelExport": true,
  "IsNonModalCRUDPage":false,
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
```

## Guide To Create A New Input File

You have to fill the fields of the JSON file for your entity. However, some of the fields must match our constants. Here is the list of them:

- MenuPosition : "main" | "admin"
  
- RelativeNamespace: Namespace of your new entity. Doesn't include your company and project name.
  
- PrimaryKeyType: "int" | "long" | "Guid"
  
- BaseClass: "Entity" | "AuditedEntity" | "CreationAuditedEntity" | "FullAuditedEntity"
  
- RelationType: Only "single" is valid for now.
  
- DuplicationNumber: A number post-fix that is added to end of variable names to prevent duplicate if there are more than one foreign key addressing to same entity. '0' means no post-fix.  


### Properties

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

or one of the ENUMs you declared in "EnumDefinitions".

## How To Run Power Tools

You can use the command below to run it:

    dotnet AspNetZeroRadTool.dll YourEntity.Json

## Notes

 - AspNetZeroRadTool folder is placed under aspnet-core folder of your solution. You have to place the json file and run the command in there.
 - Please Keep in mind that JSON file is completely **case sensitive**. 
 - Auto add-migration and update-database functions are disabled.
 - If you are working on ASP.NET Core & Angular template, after generating the entity via Power Tools, run your ***.Web.Host** project and then run "**./angular/nswag/refresh.bat**" to update **service-proxies.ts**.
