# Deleting a Metronic Theme

Metronic theme currently has 13 different themes and ASP.NET Zero includes them all. However, you might want to use only specific themes and delete some others. This document explains how to delete a theme option from ASP.NET Zero. In this document, deleting **Theme2** will be explained. You can apply the same steps to delete other theme options.

---

## .NET Part

* Go to `*.Application.Shared` project. Open `AppConsts.cs` and delete the `Theme2` field.

* Go to `*.Web.Core` project:
  * Delete `Theme2UiCustomizer.cs`
  * Open `UiThemeCustomizerFactory.cs` and delete Theme2 code parts in `GetUiCustomizerInternal` function.

* Go to `*.Core` project. Open `AppSettingProvider.cs` and delete `GetTheme2Settings` function and its call in `GetSettingDefinitions`.

---

## React Part

### 1. Delete Theme Layout Components

Go to `src/pages/admin/components/layout/themes/` folder:

* Delete the `theme2/` folder entirely (contains `Theme2Layout.tsx`, `Theme2Header.tsx`, `Theme2Brand.tsx`)

### 2. Update Theme Layout Registry

Open `src/pages/admin/components/layout/themes/index.tsx`:

* Remove the import statement:
  ```tsx
  import Theme2Layout from "./theme2/Theme2Layout";
  ```

* Remove the case from the switch statement:
  ```tsx
  case "theme2":
    return <Theme2Layout />;
  ```

### 3. Update Theme Selection Panel

Open `src/pages/admin/components/layout/theme-selection/ThemeSelectionPanel.tsx`:

* Remove `"theme2"` from the `THEMES` array:
  ```tsx
  const THEMES: string[] = [
    "default",
    // "theme2", // Remove this line
    "theme3",
    // ...
  ];
  ```

### 4. Delete UI Customization Settings Form

Go to `src/pages/admin/ui-customization/components/` folder:

* Delete `Theme2SettingsForm.tsx`

### 5. Update UI Customization Page

Open `src/pages/admin/ui-customization/index.tsx`:

* Remove the import statement:
  ```tsx
  import Theme2SettingsForm from "./components/Theme2SettingsForm";
  ```

* Remove any Theme2-related rendering logic in the component.

### 6. Delete Theme Assets (Optional)

If you want to reduce bundle size, delete the theme assets:

* Delete `metronic/themes/theme2/` folder
* Delete `public/assets/common/styles/themes/theme2/` folder
* Delete `public/assets/common/images/metronic-themes/theme2.png`

---

## Database Cleanup

If you are deleting a theme on an already published application, don't forget to delete records in the `AbpSettings` table:

* Records where `Name` equals `App.UiManagement.Theme` and `Value` equals `theme2`
* Records where `Name` starts with `Theme2.`

```sql
DELETE FROM AbpSettings WHERE Name = 'App.UiManagement.Theme' AND Value = 'theme2';
DELETE FROM AbpSettings WHERE Name LIKE 'Theme2.%';
```

---

## Summary of Files to Delete/Modify

| Action | File Path |
|--------|-----------|
| Delete | `src/pages/admin/components/layout/themes/theme2/` (entire folder) |
| Modify | `src/pages/admin/components/layout/themes/index.tsx` |
| Modify | `src/pages/admin/components/layout/theme-selection/ThemeSelectionPanel.tsx` |
| Delete | `src/pages/admin/ui-customization/components/Theme2SettingsForm.tsx` |
| Modify | `src/pages/admin/ui-customization/index.tsx` |
| Delete | `metronic/themes/theme2/` (optional) |
| Delete | `public/assets/common/styles/themes/theme2/` (optional) |
| Delete | `public/assets/common/images/metronic-themes/theme2.png` (optional) | 

