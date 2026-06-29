# Entity JSON Reference

Power Tools stores each entity definition as a JSON file in the project's `AspNetZeroRadTool` working folder. The web UI creates and edits these files for you, but you can also edit them manually for advanced scenarios or headless generation.

## When to Edit JSON Manually

Manual JSON editing is useful when you want to:

* Generate entities from automation scripts.
* Review or version entity definitions directly.
* Create an entity definition without opening the web UI.
* Adjust an option that is easier to edit as JSON.

After creating or editing the JSON file, you can generate it from the web UI or by running headless mode from the `AspNetZeroRadTool` folder:

```bash
aspnetzero-powertools YourEntity.json --no-wait --format
```

## Creating Input File

Create a JSON file in the `AspNetZeroRadTool` working folder. The file name is usually based on namespace and entity name, for example `Products.Product.json`.

JSON fields are case sensitive.

## Entity Fields

| Name | Description |
| --- | --- |
| IsRegenerate | Set `true` if this entity was generated before. |
| MenuPosition | Menu path for the generated page. Examples: `main`, `admin`, or a custom menu position. |
| RelativeNamespace | Namespace of your entity, not including the project's root namespace. |
| EntityName | Singular entity name. |
| EntityNamePlural | Plural entity name. |
| TableName | Database table name. |
| PrimaryKeyType | Primary key type. Can be `int`, `long`, `string`, or `Guid`. |
| BaseClass | Entity base class. Can be `Entity`, `AuditedEntity`, `CreationAuditedEntity`, or `FullAuditedEntity`. |
| EntityHistory | Set `true` to track history of this entity. |
| AutoMigration | Set `true` to add an EF Core migration automatically. |
| UpdateDatabase | Set `true` to update the database automatically after adding a migration. |
| CreateUserInterface | Set `true` to generate or modify UI files. |
| CreateViewOnly | Set `true` to generate a detail/view page. |
| CreateExcelExport | Set `true` to generate Excel export support. |
| CreateExcelImport | Set `true` to generate Excel import support. |
| IsNonModalCRUDPage | Set `true` when create/edit pages are full-page instead of modal-based. |
| IsMasterDetailPage | Set `true` for master-detail pages. |
| GenerateOverridableEntity | Set `true` to create extension classes that are not overwritten during re-generation. |
| GenerateUnitTest | Set `true` to generate unit tests for the application service. |
| GenerateUiTest | Set `true` to generate Playwright UI tests. |
| GenerateMobile | Set `true` to generate .NET MAUI files. |
| TemplateVariants | UI template selections used by the web UI. |
| PagePermission | Host/tenant availability. Example: `"PagePermission": { "Host": true, "Tenant": true }`. |
| Properties | Entity properties. See **Properties Table**. |
| NavigationProperties | Foreign key navigation properties. See **Navigation Properties Table**. |
| NavigationPropertyOneToManyTables | Child entities for master-detail pages. |
| EnumDefinitions | Enum definitions used by the entity. See **Enum Definitions Table**. |

## Properties Table

| Name | Description |
| --- | --- |
| Name | Property name. |
| Type | Property type. Can be `string`, `bool`, `byte`, `short`, `DateTime`, `decimal`, `double`, `Guid`, `int`, `long`, or an enum name. |
| MaxLength | Maximum string length. |
| MinLength | Minimum string length. |
| Range | Numeric range configuration. |
| Required | Whether the property is required. |
| Nullable | Whether the property is nullable. |
| Regex | Regular expression validation pattern. |
| UserInterface | UI options for list, advanced filter, and create/update forms. |

## Navigation Properties Table

| Name | Description |
| --- | --- |
| Namespace | Namespace of the foreign entity. |
| ForeignEntityName | Foreign entity name. |
| ForeignEntityNamePlural | Foreign entity plural name. |
| IdType | Foreign key type. |
| IsNullable | Whether the foreign key is nullable. |
| PropertyName | Property name that stores the foreign key. |
| DisplayPropertyName | Foreign entity property displayed in lookup UI. |
| DuplicationNumber | Suffix number used when multiple navigation properties point to the same entity. |
| RelationType | Currently `single`. |
| ViewType | Lookup UI type, such as `LookupTable`, `Dropdown`, or typeahead-supported options depending on project version. |

## One-to-Many Navigation Table

| Name | Description |
| --- | --- |
| EntityJson | Child entity JSON file name. |
| ForeignPropertyName | Property name on the child entity that stores the foreign key. |
| IsNullable | Whether the child foreign key is nullable. |
| DisplayPropertyName | Master entity property displayed on child pages. |
| ViewType | `LookupTable` or `Dropdown`. |

## Enum Definitions Table

| Name | Description |
| --- | --- |
| Name | Enum name. |
| Namespace | Enum namespace. |
| EnumProperties | Enum members. Example: `"EnumProperties": [{ "Name": "Liquid", "Value": 1 }]`. |

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
  "CreateExcelImport": false,
  "IsNonModalCRUDPage": false,
  "IsMasterDetailPage": false,
  "GenerateOverridableEntity": true,
  "GenerateUnitTest": true,
  "GenerateUiTest": false,
  "GenerateMobile": false,
  "TemplateVariants": {
    "PageTemplate": "",
    "CreateEditTemplate": "",
    "DetailTemplate": ""
  },
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
      "Namespace": "MyCompanyName.AbpZeroTemplate.Authorization.Users",
      "ForeignEntityName": "User",
      "ForeignEntityNamePlural": "Users",
      "IdType": "long",
      "IsNullable": true,
      "PropertyName": "UserId",
      "DisplayPropertyName": "Name",
      "DuplicationNumber": 0,
      "RelationType": "single",
      "ViewType": "LookupTable"
    }
  ],
  "NavigationPropertyOneToManyTables": [],
  "EnumDefinitions": [
    {
      "Name": "ProductType",
      "Namespace": "MyCompanyName.AbpZeroTemplate.Products",
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

> JSON file names and JSON field names are case sensitive.
