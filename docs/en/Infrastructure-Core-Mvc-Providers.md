# Authorization Provider

Authorization system is based on permissions. **AppPermissions** contains constants for permission names and **AppAuthorizationProvider** class defines all permissions in the system. We should define a permission here before using it in application layer.

See [authorization documentation](https://aspnetboilerplate.com/Pages/Documents/Authorization) to learn how to configure permissions.

# Feature Provider

**AppFeatureProvider** class defines features of the application for multi-tenant applications. Feature names are defined in **AppFeatures** class as contants.

See [feature management documentation](https://aspnetboilerplate.com/Pages/Documents/Feature-Management) to learn how to define and use features.

# Setting Provider

Every setting has a unique name. Setting names are defined in **AppSettings** class as constants. All settings and their default values are defined in **AppSettingProvider** class.

See [setting documentation](https://aspnetboilerplate.com/Pages/Documents/Setting-Management) to learn how to create and use settings.

# Navigation Provider

Menus are automatically generated using definitions in **AppNavigationProvider** class. We have two menus: **Main** (the main menu in the backend application) and **FrontEnd** (Main menu in front-end web site).

See [navigation documentation](https://aspnetboilerplate.com/Pages/Documents/Navigation) for more information.