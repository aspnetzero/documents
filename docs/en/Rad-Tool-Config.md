# Power Tools Configuration

The Power Tools usually does not need a manual config.  But sometimes (for example when you change the file locations) you may need to change some config so that Power Tools can work as expected.

Config.json file is located in `*YourAppFolder/aspnet-core/AspNetZeroRadTool`

*Config.json:*

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

It contains file paths. If you change any of these files path in your project. you need to change its path in config.json.



Note:

1.  It combines base paths to get MVC and Angular file paths.

   For example: It uses `CoreSrcPath + Mvc.AppNavigationProvider` for **AppNavigationProvider** path or `AngularSrcPath + Angular.AppNavigationService` for **AppNavigationService** path. 

2. It uses **AngularSrcPath** for splitted angular projects and use **AngularMergedSrcPath** for merged project. Make your changes according to the type of your project.
