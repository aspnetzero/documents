# Main Menu and Layout

Menu and Layout files located under shared folder:

<img src="D:/Github/documents/docs/en/images/ng2-layout-files-1.png" alt="Angular layout files" class="img-thumbnail" />

Main menu is defined and rendered in **side-bar.component**. You can add new menu items here. You generally relate a menu item to an Angular route. Angular routes are defined in several modules:

- app/admin/**admin-routing.module** defines routes for admin module.
- app/main/**main-routing.module** defines routes for main module.
- app/**app-routing.module** defines general routes and the default route.