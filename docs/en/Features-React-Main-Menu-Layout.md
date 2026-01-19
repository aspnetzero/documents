# Main Menu and Layout

Menu and Layout files are located under the following folders:

- Menu definition: [src/lib/navigation/appNavigation.tsx](../src/lib/navigation/appNavigation.tsx)
- Layout components: [src/pages/admin/components/layout/](../src/pages/admin/components/layout/)

ASP.NET Zero has 13 theme options and some of them use left menu while others use top menu. Both menu types get the menu definition from the `buildRawMenu` function and `useAppMenu` hook in [appNavigation.tsx](../src/lib/navigation/appNavigation.tsx). If you need to add new menu items, modify this file.

React routes are defined in:

- [src/routes/AppRouter.tsx](../src/routes/AppRouter.tsx) - defines all application routes

## Menu Item Properties

A menu item is defined using the `AppMenuItem` interface:

```typescript
export interface AppMenuItem {
  id: string;
  title: string;
  permissionName?: string;
  icon?: string;
  fontIcon?: string;
  route?: string;
  routeTemplates?: string[];
  children?: AppMenuItem[];
  external?: boolean;
  parameters?: Record<string, unknown>;
  requiresAuthentication?: boolean;
  featureDependency?: () => boolean;
}
```

| Property | Description |
|----------|-------------|
| `id` | Unique identifier for the menu item |
| `title` | Display name (use `L()` function for localization) |
| `permissionName` | Permission required to show this menu item |
| `icon` | Keenicons icon name to display |
| `fontIcon` | Font icon class name (alternative to icon) |
| `route` | React Router route path (e.g., `/app/admin/users`) |
| `routeTemplates` | Additional route patterns for active state detection |
| `children` | Child menu items for nested menus |
| `external` | If true, opens an external URL in a new tab |
| `parameters` | Additional parameters for the route |
| `requiresAuthentication` | Show to authenticated users without specific permission |
| `featureDependency` | Function to check feature dependency |

## Adding a New Menu Item

To add a new menu item, modify the `buildRawMenu` function in [appNavigation.tsx](../src/lib/navigation/appNavigation.tsx):

```typescript
export const buildRawMenu = (): AppMenuItem[] => [
  // ... existing items
  {
    id: "MyNewPage",
    title: L("MyNewPage"),
    permissionName: "Pages.MyNewPage",
    icon: "my-icon",
    route: "/app/admin/my-new-page",
  },
  // ... more items
];
```

Make sure to:
1. Add the localization key `MyNewPage` to your localization files
2. Create the corresponding route in [AppRouter.tsx](../src/routes/AppRouter.tsx)
3. Create the page component

## Feature Dependency Example

You can conditionally show menu items based on features:

```typescript
{
  id: "Chat",
  title: L("Chat"),
  icon: "message-text-2",
  route: "/app/chat",
  featureDependency: () => abp.features.isEnabled("App.ChatFeature"),
}
```

## Next

- [Edition Management](Features-React-Edition-Management)
