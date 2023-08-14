# Master Detail Tables

Master-detail tables, also known as parent-child tables, are a database design concept used to represent a one-to-many relationship between two entities. In this relationship, the data in one table (the "master" table) is related to multiple records in another table (the "detail" table).

## What is a Master-Detail Table?

The master-detail relationship is a common database design pattern and is widely used to organize related data efficiently and avoid data redundancy. It allows for easy querying and retrieval of data across both tables based on the established relationship.

Here's a breakdown of the key components:

**Master Table:** The master table contains unique records that act as the main source of information. Each record in the master table corresponds to one entity or object. For example, in a bookstore database, the master table could be "Authors," where each record represents a unique author.

**Detail Table:** The detail table contains related records that are associated with a single record in the master table. It establishes a relationship between the master table and the related data. Continuing with the bookstore example, the detail table could be "Books," where each record represents a specific book written by one of the authors in the "Authors" table. Each book record in the "Books" table would have a foreign key that references the corresponding author in the "Authors" table.

**Foreign Key:** The foreign key is a field in the detail table that stores the reference to the primary key of the corresponding record in the master table. It ensures the connection between the two tables. In our example, the "Books" table would have a foreign key column that references the "Authors" table's primary key, which might be "AuthorID."

**One-to-Many Relationship:** The relationship between the master and detail tables is typically a one-to-many relationship. It means that one record in the master table can be associated with multiple records in the detail table, but each record in the detail table can only be associated with one record in the master table.

## Create Master-Detail Table with ASP.NET ZERO Power Tools Visual Studio Extension

The ASP.NET ZERO Power Tools Visual Studio extension allows you to create a master-detail table with a few clicks. It generates the necessary code for the master and detail tables, including the foreign key relationship between the two tables.

To create a master-detail table, follow these steps:

* First, create a child entity that will be the detail table. In our example, we'll create a "Book" entity that will be the detail table for the "Author" entity, which will be the master table.
<img alt="" src="images/power-tools-book-child-entity.png" class="img-thumbnail">

* Next, create a parent entity that will be the master table. In our example, we'll create an "Author" entity that will be the master table for the "Book" entity. When creating the "Author" entity, select **Create Master-Detail Page** option and open **Navigation Property** tab.
<img alt="" src="images/power-tools-author-master-detail.png" class="img-thumbnail">

* Then, select the child entity that will be the detail table. In our example, we'll select the "Book" entity.
<img alt="" src="images/power-tools-book-master-detail.png" class="img-thumbnail">

* Click **Generate** button to generate the code for the master and detail tables.

* That's it! You've successfully created a master-detail table with ASP.NET ZERO Power Tools Visual Studio extension.

## Create Master-Detail Table Manually

* First, create a child entity that will be the detail table. For mor information about creating an entity, see [Create Entity](power-tools-creating-entity-json-file-manually.md).

* Next, create a parent entity that will be the master table.

* Then add `NavigationPropertyOneToManyTables` are to JSON file that contains child entities information.

**NavigationPropertyOneToManyTables:**

| Name                | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| EntityJson          | Child entities' json file name.(It must be located in *[YourAppPath]\aspnet-core\AspNetZeroRadTool* folder) |
| ForeignPropertyName | Property name for child entity which will store Foreign Key  |
| IsNullable          | Is nullable                                                  |
| DisplayPropertyName | Property name from base entity. It will be displayed by that property on child entities pages. |
| ViewType            | "LookupTable" or "Dropdown"                                  |

*Example JSON File*

```json
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
        "MinimumValue": 0,
        "MaximumValue": 0
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
```

## Screenshots

*Authors Page*
<img src="images/power-tools-authors-screenshot.png" alt="Rad Tool Child Entity" class="img-thumbnail" />

> **Note:** You can create list views for child entities too. Power Tools will change base entity page and child page. It will add base entity to child entity as a navigation property.

*Authors > Create New Book*
<img src="images/power-tools-authors-create-book.png" alt="Rad Tool Child Entity" class="img-thumbnail" />

*Books > Create New Book*
<img src="images/power-tools-books-create-book.png" alt="Rad Tool Child Entity" class="img-thumbnail" />

> **Note:** If you manage child entity in base entity page everything about the base entity will be automatically managed.