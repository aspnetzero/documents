# Power Tools Master Detail Tables

You can create master detail tables using Power Tools.

To create master-detail tables you can follow these steps.

1. **Creating a child entity**
   You should create necessary child entities using Power Tools 
   You can follow [that](Development-Guide-Rad-Tool.md) documentation and create an entity. 
   For example:
   <img src="images/rad-tool-master-detail-childentity1.png" alt="Rad Tool Child Entity" class="img-thumbnail" />
2. **Creating a base entity**
   * Open new entity generator. Fill your base entities' information.
   * Select **Create Master-Detail Page** then you will be able to see **Navigation Property (1-Many)** tab
     <img src="images/rad-tool-master-detail-base-entity-1.png" alt="Rad Tool Child Entity" class="img-thumbnail" />
   * Then go to that tab, Click **Add Table** and select the child table that you previously created.
     <img src="images/rad-tool-master-detail-base-entity-2.png" alt="Rad Tool Child Entity" class="img-thumbnail" />
   * Then you can generate the base entity
     <img src="images/rad-tool-master-detail-base-entity-3.png" alt="Rad Tool Child Entity" class="img-thumbnail" />

After that you can use new master detail page.

<img src="images/rad-tool-master-detail-screenshot-2.png" alt="Rad Tool Child Entity" class="img-thumbnail" />

Note: You can create list views for child entities too. Power Tools will change base entity page and child page. It will add base entity to child entity as a navigation property.

<img src="images/rad-tool-master-detail-screenshot-1.png" alt="Rad Tool Child Entity" class="img-thumbnail" />

(If you manage child entity in base entity page everything about the base entity will be automatically managed.)

<img src="images/rad-tool-master-detail-screenshot-3.png" alt="Rad Tool Child Entity" class="img-thumbnail" />
