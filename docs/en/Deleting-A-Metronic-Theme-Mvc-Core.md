# Deleting a Metronic Theme

Metronic theme currently has 12 different themes and AspNet Zero includes them all. However, you might want to use only specific themes and delete some others. This document explains how to delete a theme option from AspNet Zero. In this document, deleting **Theme2** will be explained. You can apply same steps to delete other theme options.

* Go to  `*Application.Shared` project. Open `AppConsts.cs`  and delete Theme2 field. 
* Go to `*.Web.Core` project.
  * Delete `Theme2UiCustomizer.cs`
  * Open `UiThemeCustomizerFactory.cs` and delete Theme2 code parts in `GetUiCustomizerInternal` function.
* Go to `*.Core`  project. Open `AppSettingProvider.cs` and delete `GetTheme2Settings` function
* Go to `*.Web.Mvc`
  * 
  * Go to **Areas -> AppAreaName-> Views **folder
    * Go to **Layout** folder
      * Open `_ThemeSelectionPanel.cshtml` and delete Theme2 related code parts
      * Delete ```Theme2/_Layout.cshtml``` file.
    * Go to **UICustomization** folder
      * Delete `_Theme2Settings.cshtml`	
      * Open `Index.cshml` and delete Theme2 code parts
    * Go to **Shared -> Components **folder. Delete `AppAreaNameTheme2Brand` `AppAreaNameTheme2Footer` folders.
  * Go to **wwwroot -> metronic -> themes** folder and delete theme2 folder.
  * Open `bundles.json`. Search Theme2 and delete all founded bundles.



Just note that, if you are deleting a theme on an already published application, don't forget to delete records with the name equals to "**App.UiManagement.Theme**" and name starts with "**Theme2.***" in AbpSettings table. 