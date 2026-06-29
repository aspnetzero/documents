# ASP.NET Zero Power Tools Configuration

Power Tools stores project-specific settings in the `AspNetZeroRadTool/config.json` file of your ASP.NET Zero solution. You normally manage this file from the **Setup** and **Project Settings** pages in the Power Tools web UI.

Manual edits are only needed when you customize project structure, move generated target files, or want to define advanced generation workflow commands directly in JSON.

## Where config.json is stored

The file is located under the Power Tools working folder:

```text
aspnet-core/AspNetZeroRadTool/config.json
```

If your solution uses a different root folder, use the `AspNetZeroRadTool` folder next to the selected `.sln` or `.slnx` file.

## Main Settings

Here are some key points to consider when modifying `config.json`:

1. **Company Name and Project Name:** Used to build namespaces and generated file paths.
2. **Project Type:** Supported values include `Angular`, `AngularMerged`, `Mvc`, and `React`.
3. **Project Version:** Used to enable or hide generation options that depend on ASP.NET Zero version.
4. **Application Area Name:** Used by MVC projects. The default value is usually `App`.
5. **License Code:** A valid ASP.NET Zero license code is required for code generation.
6. **Base Paths:** Paths such as `CoreSrcPath`, `AngularSrcPath`, and `AngularMergedSrcPath` are used to resolve target file locations.
7. **File Locations:** These paths tell Power Tools where to modify existing files such as `DbContext`, permissions, localization, navigation, routing, and module files.
8. **Generation Workflow:** Optional workflow steps that run after or around code generation, such as refreshing service proxies or creating bundles.

Paths are relative to the `AspNetZeroRadTool` working folder unless otherwise stated.

## Generation Workflow

The `GenerationWorkflow` array contains steps executed by the web UI generation pipeline.

Every workflow should include a generate step:

```json
{ "Type": "generate", "Label": "Code Generation" }
```

Command steps can run shell commands relative to the ASP.NET Zero project root:

```json
{
  "Type": "command",
  "Label": "Refresh Angular Proxies",
  "Command": "refresh.bat",
  "WorkingDirectory": "..\\angular\\nswag"
}
```

For long-running commands such as `dotnet run`, set `SuccessPattern` and `Timeout` so Power Tools knows when the command is ready and can continue to the next workflow step.

## Example config.json

```json
{
  "CompanyName": "MyCompanyName",
  "ProjectName": "AbpZeroTemplate",
  "ProjectType": "Angular",
  "ProjectVersion": "v15.3.0",
  "ApplicationAreaName": "App",
  "LicenseCode": "[YOURLICENSECODE]",
  "AngularSrcPath": "\\..\\..\\angular\\src\\",
  "AngularMergedSrcPath": "\\..\\src\\{{Namespace_Here}}.Web.Host\\src\\",
  "CoreSrcPath": "\\..\\src\\",
  "FormatGeneratedFiles": true,
  "GenerationWorkflow": [
    {
      "Type": "generate",
      "Label": "Code Generation"
    },
    {
      "Type": "command",
      "Label": "Run Host",
      "Command": "dotnet run",
      "WorkingDirectory": ".\\src\\MyCompanyName.AbpZeroTemplate.Web.Host",
      "SuccessPattern": "Application started",
      "Timeout": 180
    },
    {
      "Type": "command",
      "Label": "Refresh Angular Proxies",
      "Command": "refresh.bat",
      "WorkingDirectory": "..\\angular\\nswag"
    }
  ],
  "FileLocations": {
    "DbContext": "{{Namespace_Here}}.EntityFrameworkCore\\EntityFrameworkCore\\{{Project_Name_Here}}DbContext.cs",
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

Always validate JSON syntax after manual edits. Invalid JSON prevents Power Tools from loading project settings.
