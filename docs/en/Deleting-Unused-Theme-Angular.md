# Deleting Unused Theme

Its how to delete unused theme step by step.

Let's say that we are deleting Theme2.

​		***.Net Part***

* Go to  `*Application.Shared` project. Open `AppConsts.cs`  and delete Theme2 field. 

* Go to `*.Web.Core` project.
  * Delete `Theme2UiCustomizer.cs`
  * Open `GetUiCustomizerInternal.cs` and delete Theme2 code parts in `GetUiCustomizerInternal` function.
  
* Go to `*.Core`  project. Open `AppSettingProvider.cs` and delete `GetTheme2Settings` function

  

  ***Angular Part***
* Go to **src-> app -> shared -> layout** folder
  * Go to **themes** folder. Delete **theme2** folder	
  * Go to **theme-selection** folder. Open `theme-selection-panel.component.html` and delete Theme2 code parts.
  
* Go to **src-> app -> shared -> helper** folder. Open `DynamicResourceHelpers.ts` and delete Theme2 code parts.

* Go to **src -> app -> admin** folder

  * Open `admin.module.ts`  and delete Theme2 code parts.
  * Go to **ui-customization** folder
    * Open `theme2-theme-ui-settings.component.ts` and delete Theme2 code parts.
    * Open `theme2-theme-ui-settings.component.html` and delete Theme2 code parts.
    * Open `ui-customization.component.html` and delete Theme2 code parts.

* Go to **src -> app** . Open `app.module.ts` and delete Theme2 code parts.

* Go to **src -> app**. Open `app.component.html` and delete Theme2 code parts.

* Open `bundles.js` and delete Theme2 bundles.