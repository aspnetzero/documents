# Deleting Unused Theme

Its how to delete unused theme step by step.

Let's say that we are deleting Theme2.

* Go to  `*Application.Shared` project. Open `AppConsts.cs`  and delete Theme2 field. 

* Go to `*.Web.Core` project.
  * Delete `Theme2UiCustomizer.cs`
  * Open `UiThemeCustomizerFactory.cs` and delete Theme2 code parts in `GetUiCustomizerInternal` function.
* Go to `*.Core`  project. Open `AppSettingProvider.cs` and delete `GetTheme2Settings` function
* Go to `*.Web.Mvc`
  * Go to **Areas -> App -> Views **folder
    * Go to **UICustomization** folder
      * Delete `_Theme2Settings.cshtml`	
      * Open `Index.cshml` and delete Theme2 code parts
    * Go to **Shared -> Components **folder. Delete `AppAreaNameTheme2Brand` `AppAreaNameTheme2Footer` folders.
  * Go to **wwwroot -> metronic -> themes** folder and delete theme2 folder.
  * Open `bundles.json`. Search Theme2 and delete all founded bundles.