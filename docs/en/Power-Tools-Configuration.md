# ASP.NET ZERO Power Tools Configuration

The `config.json` file provided in your ASP.NET Zero project contains essential file paths and configuration settings used by the Power Tools. While the Power Tools **typically do not require manual configuration**, there are instances where you **might need** to modify the configuration to adapt to changes in file locations or project structure. In such cases, updating the `config.json` becomes necessary to ensure the Power Tools function correctly.

Here are some key points to consider when modifying the `config.json` file:

1. **Company Name and Project Name:** You may need to update the **CompanyName** and **ProjectName** fields with appropriate values to match your project's details.

2. **Project Type and Version:** Set the **ProjectType** to **Angular** or **Mvc** and specify the appropriate **ProjectVersion**.

3. **Application Area Name:** Ensure the **ApplicationAreaName** matches the relevant area name in your project.

4. **License Code:** Replace **[YOURLICENSECODE]** with the valid license code for your project.

5. **File Locations:** This section contains various file paths used by the Power Tools for different functionalities. If you change any of these file locations in your project, you need to update the corresponding paths in the `config.json` file accordingly. For example, if you move the DbContext file to a different location, update the **DbContext** path to reflect the new location.

6. **Base Paths:** The `config.json` file uses base paths (e.g., `AngularSrcPath`, `AngularMergedSrcPath`, `CoreSrcPath`) to build the final file paths. Make sure to set these base paths correctly according to your project's structure.

7. **Angular Project Type:** Depending on whether your Angular project is split into multiple projects or merged into a single project, use the appropriate base path (`AngularSrcPath` for split projects or `AngularMergedSrcPath` for merged projects) for the Angular-related file paths.

Always **double-check** the **changes** you make in the config.json file to avoid any errors or inconsistencies. It's crucial to keep the configuration updated to ensure the Power Tools can accurately locate the required files and function as expected with your project's structure.

*Example config.json*
```json
{
  "CompanyName": "",
  "ProjectName": "[YOURPROJECTNAME]",
  "ProjectType": "Angular",
  "ProjectVersion": "v8.0.0",
  "ApplicationAreaName": "App",
  "LicenseCode": "[YOURLICENSECODE]",
  "AngularSrcPath": "\\..\\..\\angular\\src\\",
  "AngularMergedSrcPath": "\\..\\src\\{{Namespace_Here}}.Web.Host\\src\\",
  "CoreSrcPath": "\\..\\src\\",
  "FormatGeneratedFiles": true,
  "FileLocations": {
    "DbContext": "{{Namespace_Here}}.EntityFrameworkCore\\EntityFrameworkCore\\{{Project_Name_Here}}DbContext.cs",
    "CustomDtoMapper": "{{Namespace_Here}}.Application\\CustomDtoMapper.cs",
    "AppAuthorizationProvider": "{{Namespace_Here}}.Core\\Authorization\\AppAuthorizationProvider.cs",
    "EntityHistoryHelper": "{{Namespace_Here}}.Core\\EntityHistory\\EntityHistoryHelper.cs",
    "AppPermissions": "{{Namespace_Here}}.Core\\Authorization\\AppPermissions.cs",
    "LocalizationFile": "{{Namespace_Here}}.Core\\Localization\\{{Project_Name_Here}}\\{{Project_Name_Here}}.xml",
    "EntityFrameWorkProjectFolder": "{{Namespace_Here}}.EntityFrameworkCore",
    "Mvc": {
      "AppNavigationProvider": "{{Namespace_Here}}.Web.Mvc\\Areas\\{{App_Area_Name_Here}}\\Startup\\{{App_Area_Name_Here}}NavigationProvider.cs",
      "AppPageNames": "{{Namespace_Here}}.Web.Mvc\\Areas\\{{App_Area_Name_Here}}\\Startup\\{{App_Area_Name_Here}}PageNames.cs",
      "BundleConfig": "{{Namespace_Here}}.Web.Mvc\\bundles.json"
    },
    "Angular": {
      "AppNavigationService": "app\\shared\\layout\\nav\\app-navigation.service.ts",
      "ServiceProxies": "shared\\service-proxies\\service-proxy.module.ts",
      "Module": "app\\{{menu_Position_Here}}\\{{menu_Position_Here}}.module.ts",
      "RoutingModule": "app\\{{menu_Position_Here}}\\{{menu_Position_Here}}-routing.module.ts"
    }
  }
}
```