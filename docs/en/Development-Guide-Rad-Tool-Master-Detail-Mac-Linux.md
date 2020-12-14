# Power Tools Master Detail Tables

You can create master detail tables using Power Tools.

To create master-detail tables you can follow that steps.

1. **Creating a child entity**
   You should create necessary child entities using Power Tools 
   You can follow [that](Development-Guide-Rad-Tool-Mac-Linux.md) documentation and create an entity. 
   For example:

   ``````json
   {
     "IsRegenerate": false,
     "MenuPosition": "main",
     "RelativeNamespace": "ChildNamespace1",
     "EntityName": "Child",
     "EntityNamePlural": "Childs",
     "TableName": "Childs",
     "PrimaryKeyType": "int",
     "BaseClass": "Entity",
     "EntityHistory": false,
     "AutoMigration": true,
     "UpdateDatabase": true,
     "CreateUserInterface": true,
     "CreateViewOnly": true,
     "CreateExcelExport": true,
     "IsNonModalCRUDPage": false,
     "IsMasterDetailPage": false,
     "PagePermission": {
       "Host": true,
       "Tenant": true
     },
     "Properties": [
       {
         "Name": "ChildProp1",
         "Type": "string",
         "MaxLength": -1,
         "MinLength": -1,
         "Range": {
           "IsRangeSet": false,
           "MinimumValue": 0.0,
           "MaximumValue": 0.0
         },
         "Required": false,
         "Nullable": false,
         "Regex": "",
         "UserInterface": {
           "AdvancedFilter": true,
           "List": true,
           "CreateOrUpdate": true
         }
       }
     ],
     "NavigationProperties": [
       {
         "Namespace": "Zerov921CoreAngularDemo.Authorization.Users",
         "ForeignEntityName": "User",
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
     "EnumDefinitions": [],
     "DbContext": null
   }
   ``````

2. **Creating a base entity**

   * You can follow [that](Development-Guide-Rad-Tool-Mac-Linux.md) documentation and prepare an entity. 

   * Then add `NavigationPropertyOneToManyTables` are to JSON file that contains child entities' information.

     **NavigationPropertyOneToManyTables:**

     | Name                | Description                                                  |
     | ------------------- | ------------------------------------------------------------ |
     | EntityJson          | Child entities' json file name.(It must be located in *[YourAppPath]\aspnet-core\AspNetZeroRadTool* folder) |
     | ForeignPropertyName | Property name for child entity which will store Foreign Key  |
     | IsNullable          | Is nullable                                                  |
     | DisplayPropertyName | Property name from base entity. It will be displayed by that property on child entities pages. |
     | ViewType            | "LookupTable" or "Dropdown"                                  |


     For Example:
    
     ``````json
     {
       "IsRegenerate": false,
       "MenuPosition": "main",
       "RelativeNamespace": "BaseNamespace",
       "EntityName": "BaseEntity",
       "EntityNamePlural": "BaseEntities",
       "TableName": "BaseEntities",
       "PrimaryKeyType": "int",
       "BaseClass": "Entity",
       "EntityHistory": false,
       "AutoMigration": true,
       "UpdateDatabase": true,
       "CreateUserInterface": true,
       "CreateViewOnly": true,
       "CreateExcelExport": true,
       "IsNonModalCRUDPage": false,
       "IsMasterDetailPage": true,
       "PagePermission": {
         "Host": true,
         "Tenant": true
       },
       "Properties": [
         {
           "Name": "BaseProp1",
           "Type": "string",
           "MaxLength": -1,
           "MinLength": -1,
           "Range": {
             "IsRangeSet": false,
             "MinimumValue": 0.0,
             "MaximumValue": 0.0
           },
           "Required": false,
           "Nullable": false,
           "Regex": "",
           "UserInterface": {
             "AdvancedFilter": true,
             "List": true,
             "CreateOrUpdate": true
           }
         }
       ],
       "NavigationProperties": [
         {
           "Namespace": "Abp.Organizations",
           "ForeignEntityName": "OrganizationUnit",
           "IdType": "long",
           "IsNullable": true,
           "PropertyName": "OrganizationUnitId",
           "DisplayPropertyName": "DisplayName",
           "DuplicationNumber": 0,
           "RelationType": "single",
           "ViewType": "LookupTable"
         }
       ],
       "NavigationPropertyOneToManyTables": [
           {
               "EntityJson": "ChildNamespace1.Child.json",
               "ForeignPropertyName": "BaseEntityId",
               "IsNullable": "True",
               "DisplayPropertyName": "BaseProp1",
               "ViewType": "LookupTable"
           }
       ],
       "EnumDefinitions": [],
       "DbContext": null
     }
     ``````

   * Then you can generate the base entity
     Check: https://docs.aspnetzero.com/en/common/latest/Development-Guide-Rad-Tool-Mac-Linux#how-to-run-power-tools

After that you can use new master detail page.

<img src="images/rad-tool-master-detail-screenshot-2.png" alt="Rad Tool Child Entity" class="img-thumbnail" />

Note: You can create list views for child entities too. Power Tools will change base entity page and child page. It will add base entity to child entity as a navigation property.

<img src="images/rad-tool-master-detail-screenshot-1.png" alt="Rad Tool Child Entity" class="img-thumbnail" />

(If you manage child entity in base entity page everything about the base entity will be automatically managed.)

<img src="images/rad-tool-master-detail-screenshot-3.png" alt="Rad Tool Child Entity" class="img-thumbnail" />
