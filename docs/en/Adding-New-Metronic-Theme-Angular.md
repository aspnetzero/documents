# Adding New Metronic Theme

Metronic theme currently has 12 different themes and AspNet Zero includes them all. However, you might want to add a new theme option designed by your team to those options. This document explains step by step to add a new theme option to AspNet Zero. Just note that, the added theme must be a Metronic theme or at least it must be compatible with Metronic.

Rest of the document will use **ThemeX** as the new theme name.

##### 		*.Net Part*

* Go to  `*Application.Shared` project, open `AppConsts.cs`  and add new field named ThemeX. 

* Go to `*.Web.Core` project.

  - Create new UICustomizer named `ThemeXUiCustomizer.cs` . Copy `ThemeDefaultUiCustomizer.cs` into  `ThemeXUiCustomizer.cs` and change necessary settings. (It has setting methods, If your new ThemeX also have that settings keep them otherwise delete them)

  - Open `UiThemeCustomizerFactory.cs` and add ThemeX code parts in `GetUiCustomizerInternal` function.

    ```csharp
    ...
    if (theme.Equals(AppConsts.ThemeX, StringComparison.InvariantCultureIgnoreCase))
    {
          return _serviceProvider.GetService<ThemeXUiCustomizer>();
    }
    ...
    ```

* Go to `*.Core`  project. Open `AppSettingProvider.cs` 

  - Add a method named `GetThemeXSettings`  which returns ThemeX settings.

  - Call it in `GetSettingDefinitions` function

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



##### *Angular Part*

* Go to  **src-> app -> shared -> layout** folder
  * Go to **themes** folder.
    * Create a folder named `themeX` and go to **themeX** folder.
      * Create components named  `themeX-brand` and `themeX-layout`
      * Copy their body from default theme components (from `default-brand` and `default-layout`) and change needed changes.
      * Create `ThemeXThemeAssetContributor` and copy its content from `DefaultThemeAssetContributor` .This class returns additional assets so make needed changes for your new themeX.
  * Go to **theme-selection** folder. Open `theme-selection-panel.component.html` and add themeX to list.
* Go to **src -> shared -> helpers** and open `ThemeAssetContributorFactory.ts`. Add `ThemeXThemeAssetContributor` to `getCurrent` function

```csharp
...
if (theme === 'themeX') {
	return new ThemeXThemeAssetContributor().getAssetUrls();
}
...
```

* Go to **src -> app -> admin** folder

  * Go to **ui-customization** folder.

    * Create `themex-theme-ui-settings` component. Copy its content from `default-theme-ui-settings` component. This is where you select UI settings, If your new ThemeX also have that settings keep them otherwise delete them and add what is needed.

    * Open `ui-customization.component.html` and add your component.

      

      ```html
      <tab *ngFor="let themeSetting of themeSettings" [active]="themeSetting.theme == currentThemeName">
       <!--...-->
          <themex-theme-ui-settings *ngIf="themeSetting.theme == 'themeX'" [settings]="themeSetting"></themex-theme-ui-settings>
      </tab>
      ```

  * Open `ui-customization.module.ts` and add `ThemeXThemeUiSettingsComponent`to declarations.

* Go to **src -> app** . Open `app.module.ts` and add `ThemeXLayoutComponent` and `ThemeXBrandComponent` to declarations.

* Go to **src -> app**. Open `app.component.html` and add ThemeX code part.

  ```html
  <div [ngClass]="{'subscription-bar-visible': subscriptionStatusBarVisible()}">
      <!--...-->
      <themex-layout *ngIf="theme=='themeX'"></themex-layout>
  </div>
  ```

* If your theme uses dynamic bundles, open the `bundle.js` and add your bundles.

  _ASP.NET Zero has custom CSS file for datepicker in Angular version ([bs-datepicker.css](https://github.com/aspnetzero/aspnet-zero-core/blob/dev/angular/src/assets/ngx-bootstrap/bs-datepicker.css)). If the default design is not compatible with your theme, you can change the related CSS file._
