# Deleting a Metronic Theme

Metronic theme currently has 12 different themes and AspNet Zero includes them all. However, you might want to use only specific themes and delete some others. This document explains how to delete a theme option from AspNet Zero. In this document, deleting **Theme2** will be explained. You can apply same steps to delete other theme options.

​		***.Net Part***

* Go to  `*Application.Shared` project. Open `AppConsts.cs`  and delete Theme2 field. 

* Go to `*.Web.Core` project.
  * Delete `Theme2UiCustomizer.cs`
  * Open `UiThemeCustomizerFactory.cs` and delete Theme2 code parts in `GetUiCustomizerInternal` function.
  
* Go to `*.Core`  project. Open `AppSettingProvider.cs` and delete `GetTheme2Settings` function

  

  ***Angular Part***
  
* Go to **src-> app -> shared -> layout** folder
  * Go to **themes** folder. Delete **theme2** folder	
  * Go to **theme-selection** folder. Open `theme-selection-panel.component.html` and delete Theme2 code parts.
  
* Go to **src-> app -> shared -> helper** folder. Open `DynamicResourceHelpers.ts` and delete Theme2 code parts.

* Go to **src -> app -> admin** folder

  * Open `admin.module.ts`  and delete Theme2 code parts.
  * Go to **ui-customization** folder
    * Delete `theme2-theme-ui-settings.component.ts`.
    * Delete `theme2-theme-ui-settings.component.html`.
    * Open `ui-customization.component.html` and delete Theme2 code parts.

* Go to **src -> app** . Open `app.module.ts` and delete Theme2 code parts.

* Go to **src -> app**. Open `app.component.html` and delete Theme2 code parts.

* Open `bundles.js` and delete Theme2 bundles.



Just note that, if you are deleting a theme on an already published application, don't forget to delete records with the name equals to "**App.UiManagement.Theme**" and name starts with "**Theme2.***" in AbpSettings table. 
