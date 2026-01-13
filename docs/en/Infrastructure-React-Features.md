# Features

You can use the global `abp.features` object to check tenant features in your React components.

## Usage

```tsx
// Check if a feature is enabled
if (abp.features.isEnabled("App.ChatFeature")) {
  // Feature is enabled
}

// Get feature value
const maxUserCount = abp.features.getValue("App.MaxUserCount");
```

## Using in Menu Items

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

Features are defined on the server side in the `*.Core` project. See feature management [documentation](https://aspnetboilerplate.com/Pages/Documents/Feature-Management) for more.
