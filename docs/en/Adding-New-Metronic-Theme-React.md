# Adding New Metronic Theme

Metronic theme currently has 13 different themes and ASP.NET Zero includes them all. However, you might want to add a new theme option designed by your team to those options. This document explains step by step how to add a new theme option to ASP.NET Zero React UI. Note that the added theme must be a Metronic theme or at least be compatible with Metronic.

The rest of this document will use **ThemeX** as the new theme name.

---

## .NET Part

* Go to `*.Application.Shared` project, open `AppConsts.cs` and add a new field named `ThemeX`.

* Go to `*.Web.Core` project:

  - Create a new UI Customizer named `ThemeXUiCustomizer.cs`. Copy `ThemeDefaultUiCustomizer.cs` into `ThemeXUiCustomizer.cs` and change necessary settings. (It has setting methods; if your new ThemeX also has those settings, keep them; otherwise, delete them.)

  - Open `UiThemeCustomizerFactory.cs` and add the ThemeX code in the `GetUiCustomizerInternal` function:

    ```csharp
    if (theme.Equals(AppConsts.ThemeX, StringComparison.InvariantCultureIgnoreCase))
    {
        return _serviceProvider.GetService<ThemeXUiCustomizer>();
    }
    ```

* Go to `*.Core` project. Open `AppSettingProvider.cs`:

  - Add a method named `GetThemeXSettings` which returns ThemeX settings.

  - Call it in the `GetSettingDefinitions` function:

    ```csharp
    return GetHostSettings().Union(GetTenantSettings()).Union(GetSharedSettings())
        // theme settings
        .Union(GetDefaultThemeSettings())
        .Union(GetTheme2Settings())
        // ... other themes ...
        .Union(GetTheme13Settings())
        .Union(GetThemeXSettings()); // add ThemeXSettings
    ```

---

## React Part

### 1. Add Theme Assets

Add your theme's CSS bundle files to the `metronic/themes/` folder:

```
metronic/themes/themeX/
├── css/
│   └── style.bundle.css
└── plugins/
    └── global/
        └── plugins.bundle.css
```

Also add a customization CSS file:

```
public/assets/common/styles/themes/themeX/
└── metronic-customize.css
```

### 2. Create Theme Layout Components

Go to `src/pages/admin/components/layout/themes/` folder:

1. Create a new folder named `themeX/`
2. Create the following React components inside the folder:
   - `ThemeXLayout.tsx` - Main layout component
   - `ThemeXHeader.tsx` - Header component
   - `ThemeXBrand.tsx` - Brand/logo component

Copy the contents from the default theme components (`default/DefaultLayout.tsx`, `default/DefaultHeader.tsx`, `default/DefaultBrand.tsx`) and modify as needed.

**Example `ThemeXLayout.tsx`:**

```tsx
import React, { useEffect } from "react";
import { Outlet } from "react-router-dom";
import ThemeXHeader from "./ThemeXHeader";
import { Footer } from "../../Footer";
import { useTheme } from "@/hooks/useTheme";
import { SidebarMenu } from "../../sidebar-menu/SidebarMenu";
import ThemeXBrand from "./ThemeXBrand";

const ThemeXLayout: React.FC = () => {
  const { containerClass } = useTheme();

  useEffect(() => {
    document.body.classList.add("app-themeX");
    return () => {
      document.body.classList.remove("app-themeX");
    };
  }, []);

  return (
    // ... your layout JSX
  );
};

export default ThemeXLayout;
```

### 3. Register the Theme Layout

Open `src/pages/admin/components/layout/themes/index.tsx` and add your theme:

1. Import the new layout component:

   ```tsx
   import ThemeXLayout from "./themeX/ThemeXLayout";
   ```

2. Add the case to the switch statement in `ThemedLayout`:

   ```tsx
   const ThemedLayout: React.FC = () => {
     const activeThemeName = useSelector(
       (s: RootState) => s.ui.activeThemeName || "default",
     );

     switch (activeThemeName) {
       // ... existing cases ...
       case "themeX":
         return <ThemeXLayout />;
       case "default":
       default:
         return <DefaultLayout />;
     }
   };
   ```

### 4. Add Theme to Selection Panel

Open `src/pages/admin/components/layout/theme-selection/ThemeSelectionPanel.tsx` and add your theme to the `THEMES` array:

```tsx
const THEMES: string[] = [
  "default",
  "theme2",
  "theme3",
  // ... other themes ...
  "theme13",
  "themeX", // Add your new theme
];
```

### 5. Create UI Customization Settings Form

Go to `src/pages/admin/ui-customization/components/` folder:

1. Create `ThemeXSettingsForm.tsx` by copying from `DefaultThemeSettingsForm.tsx`
2. Modify the form fields based on your theme's available settings

**Example `ThemeXSettingsForm.tsx`:**

```tsx
import React from "react";
import { Form, Select, Switch } from "antd";
import type { ThemeSettingsDto } from "@/api/generated/service-proxies";
import L from "@/lib/L";

interface Props {
  settings: ThemeSettingsDto;
  onSave: (values: ThemeSettingsDto) => void;
}

const ThemeXSettingsForm: React.FC<Props> = ({ settings, onSave }) => {
  const [form] = Form.useForm();

  // ... form implementation
};

export default ThemeXSettingsForm;
```

### 6. Register the Settings Form

Open `src/pages/admin/ui-customization/index.tsx`:

1. Import your settings form:

   ```tsx
   import ThemeXSettingsForm from "./components/ThemeXSettingsForm";
   ```

2. Add the form to the theme panel rendering logic (in the switch or conditional rendering section).

### 7. Add Theme Preview Image

Add a preview image for your theme:

```
public/assets/common/images/metronic-themes/themeX.png
```

This image is displayed in the UI Customization page for theme selection.

### 8. Add Localization

Add localization entries for your theme name in the localization files:

```json
{
  "Theme_ThemeX": "Theme X"
}
```

---

## Summary of Files to Create/Modify

| Action | File Path |
|--------|-----------|
| Create | `metronic/themes/themeX/css/style.bundle.css` |
| Create | `metronic/themes/themeX/plugins/global/plugins.bundle.css` |
| Create | `public/assets/common/styles/themes/themeX/metronic-customize.css` |
| Create | `src/pages/admin/components/layout/themes/themeX/ThemeXLayout.tsx` |
| Create | `src/pages/admin/components/layout/themes/themeX/ThemeXHeader.tsx` |
| Create | `src/pages/admin/components/layout/themes/themeX/ThemeXBrand.tsx` |
| Modify | `src/pages/admin/components/layout/themes/index.tsx` |
| Modify | `src/pages/admin/components/layout/theme-selection/ThemeSelectionPanel.tsx` |
| Create | `src/pages/admin/ui-customization/components/ThemeXSettingsForm.tsx` |
| Modify | `src/pages/admin/ui-customization/index.tsx` |
| Create | `public/assets/common/images/metronic-themes/themeX.png` |

