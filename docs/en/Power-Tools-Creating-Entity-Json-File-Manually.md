# Creating Entity Json File Manually

In this document, we will explain how to use **ASP.NET Zero Power Tools** without the Visual Studio extension.

Purpose of the **ASP.NET Zero Power Tools VS Extension** is to create an input file. So, in order to use it without extension, input file needs to be created manually. 

## Creating Input File

For creating JSON file manually, you need to learn fields of the JSON file (configuration for entity).

Properties are written as an array in JSON file. Add an object to that array for every property of your entity. There will be some unnecessary fields depending on property type. For example, you don't have to set regular expression for a numeric property or don't have to set range for a string.

You have to fill the fields of the JSON file for your entity. However, some of the fields must match our constants. 

A property should be one of those types:

* bool
* byte
* short
* DateTime
* decimal
* double
* Guid
* int
* long
* string

or one of the ENUMs you declared in **EnumDefinitions**.

You can find specification of the JSON file fields in the table below;

## Fields Table

| Name | Description |
| --- | --- |
| IsRegenerate | Set `true` if you have generated this entity before. |
| MenuPosition | `main` or `admin` |
| RelativeNamespace | Namespace of your entity (not including project's namespace) |
| EntityName | Entity Name |
| EntityNamePlural | Entity Name Plural |
| TableName | Database Table Name (might be same with plural name) |
| PrimaryKeyType | Type of primay key. <br />Can be `int`, `long`, `string`, `Guid` |
| BaseClass | Base class of your entity. <br />Can be `Entity`, `AuditedEntity`, `CreationAuditedEntity`, `FullAuditedEntity` |
| EntityHistory | Set `true` to track history of this entity. |
| AutoMigration | `true` add-migration automatically, false do not add migration (you need to add migration manually) |
| UpdateDatabase | `true` update-database automatically, false do not update-database (you need to update-database manually) |
| CreateUserInterface | `true` creates/modifies ui layer files |
| CreateViewOnly | `true` creates a view-only modal in actions button in table of your entity in ui |
| CreateExcelExport | `true` adds excel report button in ui |
| IsNonModalCRUDPage | `true` creates non-modal pages. |
| PagePermission | Multitenancy<br />`"PagePermission":{"Host": [ISHOSTALLOWED],"Tenant":[ISTENANTALLOWED]}` |
| Properties | Properties of your entity. See **Properties Table** for more. |
| NavigationProperties | Navigation properties of your entity. See **NavigationProperties Table** for more. |
| EnumDefinitions | Enum definitions you use on your entity. See **EnumDefinitions Table** for more. |

## Properties Table(Array):

| Name | Description |
| --- | --- |
| Name | Property Name |
| Type | Type of property. <br />Can be string, bool, byte, short, DateTime, decimal, double, Guid, int, long, enum |
| MaxLength | If type is string max length of string |
| MinLength | If type is string min length of string |
| Range | If type can have range value range of property<br />"Range": {"IsRangeSet": [ISRANGESET],"MinimumValue": [MINVAL],"MaximumValue": [MAXVAL]} |
| Required | Is property required |
| Nullable | Is property nullable |
| Regex | specifies the regex that this property should match |
| UserInterface | Will this property be listed, have a filter and editable in ui?<br />"UserInterface": {"List": true,"AdvancedFilter": true,"CreateOrUpdate": true} |

## NavigationProperties Table(Array):

| Name | Description |
| --- | --- |
| Namespace | Namespace of entity |
| ForeignEntityName | Foreign Entity Name |
| ForeignEntityNamePlural | Foreign Entity Name Plural |
| IdType | Type of Foreign Key. See Entity -> PrimaryKeyType |
| IsNullable | Is nullable |
| PropertyName | Property name (Property name for that entity which will store Foreign Key) |
| DisplayPropertyName | Property name of foreign entity. It will be displayed by that property on pages. |
| DuplicationNumber | If you have two navigation property that navigates to same foreign entity, number them starting from 1. If not, just skip this. |
| RelationType | single is the only option. |

## EnumDefinitions Table:

| Name | Description |
| --- | --- |
| Name | Name |
| Namespace | Namespace |
| EnumProperties | Properties <br />"EnumProperties":[{"Name":"[PROPERYNAME]","Value":[PROPERYVALUE]}] |

## Example JSON Input

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
  "IsNonModalCRUDPage": false,
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

> Note: Please Keep in mind that JSON file is completely **case sensitive**.
