# Adding New Metronic Theme

Its how to add new theme to your project step by step.

Let's say that we are adding theme named "ThemeX"

##### 		*.Net Part*

* Go to  `*Application.Shared` project. Open `AppConsts.cs`  and add new field named ThemeX. 

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

  - Add function  named `GetThemeXSettings`  which return ThemeX settings. (Where you turn back settings. If you did same changes in `ThemeDefaultUiCustomizer.cs` , return that settings too.)

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
    * Create folder named `themeX`.
    * Create components named  `themeX-brand` and `themeX-layout`
    * Copy their body from default theme components (from `default-brand` and `default-layout`) and change needed changes.
    * Create `ThemeXThemeAssetContributor` and copy its content from `DefaultThemeAssetContributor` .This class returns additional assets so change needed changes for your new themeX.
  * Go to **theme-selection** folder. Open `theme-selection-panel.component.html` and add themeX to list.
* Go to **src-> app -> shared -> helper**  and open `DynamicResourceHelpers.ts`. Add `ThemeXThemeAssetContributor` to `getAdditionalThemeAssets` function

```csharp
...
if (theme === 'themeX') {
	return new ThemeXThemeAssetContributor().getAssetUrls();
}
...
```

* Go to **src -> app -> admin** folder

  * Go to **ui-customization** folder.

    * Create `themex-theme-ui-settings` component. Copy its content from `default-theme-ui-settings` component. This is where you select ui settings, If your new ThemeX also have that settings keep them otherwise delete them and add what is needed.

    * Open `ui-customization.component.html` and add your component.

      

      ```html
      <tab *ngFor="let themeSetting of themeSettings" [active]="themeSetting.theme == currentThemeName">
       <!--...-->
          <themex-theme-ui-settings *ngIf="themeSetting.theme == 'themeX'" [settings]="themeSetting"></themex-theme-ui-settings>
      </tab>
      ```

  * Open `admin.module.ts` and add `ThemeXThemeUiSettingsComponent`to declarations.

* Go to **src -> app** . Open `app.module.ts` and add `ThemeXLayoutComponent` and `ThemeXBrandComponent` to declarations.

* Go to **src -> app**. Open `app.component.html` and add ThemeX code part.

  ```html
  <div [ngClass]="{'subscription-bar-visible': subscriptionStatusBarVisible()}">
      <!--...-->
      <themex-layout *ngIf="theme=='themeX'"></themex-layout>
  </div>
  ```

* If your theme use dynamic bundles. Open `bundle.js` add your bundles.