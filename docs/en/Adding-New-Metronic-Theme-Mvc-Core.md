# Adding New Metronic Theme

Metronic theme currently has 12 different themes and AspNet Zero includes them all. However, you might want to add a new theme option designed by your team to those options. This document explains step by step to add a new theme option to AspNet Zero. Just note that, the added theme must be a Metronic theme or at least it must be compatible with Metronic.

Rest of the document will use **ThemeX** as the new theme name.

* Go to  `*Application.Shared` project. Open `AppConsts.cs`  and add a new field named ThemeX. 

* Go to `*.Web.Core` project.

  - Create new UICustomizer named `ThemeXUiCustomizer.cs` . Copy `ThemeDefaultUiCustomizer.cs` into  `ThemeXUiCustomizer.cs` and change necessary settings. (It has setting methods, If your new ThemeX also have that settings keep them otherwise delete them)

  - Open `UiThemeCustomizerFactory.cs` and add ThemeX code parts in `GetUiCustomizerInternal` method.

    ```csharp
    ...
    if (theme.Equals(AppConsts.ThemeX, StringComparison.InvariantCultureIgnoreCase))
    {
          return _serviceProvider.GetService<ThemeXUiCustomizer>();
    }
    ...
    ```

* Go to `*.Core`  project. Open `AppSettingProvider.cs` 

  *  Add a function named `GetThemeXSettings` which returns ThemeX settings. 

  * Call it in `GetSettingDefinitions` function

    ```csharp
    ... 
    return GetHostSettings().Union(GetTenantSettings()).Union(GetSharedSettings())
                    // theme settings
                    .Union(GetDefaultThemeSettings())
                    .Union(GetTheme2Settings())
                    .Union(GetTheme3Settings())
                    .Union(GetTheme4Settings())
                    .Union(GetTheme5Settings())
                    .Union(GetTheme6Settings())
                    .Union(GetTheme7Settings())
                    .Union(GetTheme8Settings())
                    .Union(GetTheme9Settings())
                    .Union(GetTheme10Settings())
                    .Union(GetTheme11Settings())
                    .Union(GetTheme12Settings())
                    .Union(GetThemeXSettings());//add ThemeXSettings
    ...
    ```

* Go to `*.Web.Mvc` project

  * Go to **Areas -> App -> Views** folder.

    * Create new view named `_ThemeXSettings.cshtml` . Copy `_DefaultSettings.chtml` content into `_ThemeXSettings.cshtml`. (Make same changes which you did in `ThemeXUiCustomizer`)
    * Open `Index.cshtml` and add your new theme.

    ```html
     <!--Add tab-->
    <ul class="nav nav-pills" role="tablist">
     ...
    <!----Add These Part---->
     <li class="nav-item">
        <a href="#themeX" class="nav-link @(Model.Theme == AppConsts.ThemeX ? "active" : "")" data-toggle="tab" role="tab">
            <img src="@(ApplicationPath)Common/Images/metronic-themes/themeX.png" width="150" /><!--Your theme image path-->
            <span class="theme-name">@L("Theme_" + AppConsts.ThemeX.ToPascalCase())</span>
        </a>
    </li>
    <!-------->
     <ul>
     <div class="tab-content">
     ...
    <!----Add These Part---->
    <div id="themeX" class="tab-pane theme-selection @(Model.Theme == AppConsts.ThemeX ? "active show" : "")">
        @await Html.PartialAsync("_ThemeXSettings", Model.GetThemeSettings(AppConsts.ThemeX))
     </div>
    <!-------->
    </div>
    ```

  * Go to **Shared -> Components **folder.

    Copy **AppAreaNameDefaultBrandViewComponent** and **AppAreaNameDefaultFooterViewComponent ** folders. These folders has 2 component Change their name and necessary parts.

  * Go to **wwwroot -> metronic -> themes** and create folder named themeX.  Copy default folder content here and change what you need (change css, js as you wish).

  * Open `bundle.json` and add new bundles that you add **wwwroot -> metronic -> themes -> themeX** 

