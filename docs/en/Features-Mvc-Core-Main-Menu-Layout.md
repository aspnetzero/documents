# Main Menu

Application's main menu is defined in **AppNavigationProvider** class. See ABP's [navigation documentation](https://aspnetboilerplate.com/Pages/Documents/Navigation)
to have a deep understanding on creating menus. When you add a new menu item, it's automatically rendered in the layout.

# Layout

Layout of the application is located under **Areas/App/Views/Layout** folder. It uses **Components** for **header**, **footer** and **sidebar**:

<img src="images/app-layout-core-1.png" alt="Application layout" class="img-thumbnail" />

Main menu is rendered in **menu** component. Layout also highly uses bundling & minification (see the section below) system for script and style includes.

## Next

- [Edition Management](Features-Mvc-Core-Edition-Management)