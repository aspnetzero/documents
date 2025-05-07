# How to Add a New Language in an ASP.NET Zero Angular Application

## Introduction

This document provides step-by-step instructions for adding a new localization (language) in your ASP.NET Zero Angular application. Localization allows users to interact with the application in their preferred language, thereby improving usability and accessibility. For more details on localization in ASP.NET Boilerplate, refer to the official documentation: [ASP.NET Boilerplate Localization](https://aspnetboilerplate.com/Pages/Documents/Localization).

## Steps to Add a New Language

### 1. Register the New Language in the Backend

To support a new language in the backend, you need to register it in the `DefaultLanguagesCreator.cs` file located in the `YourProjectName.Core` project.

#### Modify DefaultLanguagesCreator.cs

In this step, you need to add the new language to the list of initial languages registered in the system. This ensures that the application recognizes the new language and makes it available for selection. Modify the `GetInitialLanguages` method to include the new language code, display name, and flag icon.

**DefaultLanguagesCreator.cs**
```c#
public class DefaultLanguagesCreator
{
    public static List<ApplicationLanguage> InitialLanguages => GetInitialLanguages();

    private readonly AbpZeroTemplateDbContext _context;

    private static List<ApplicationLanguage> GetInitialLanguages()
    {
        var tenantId = AbpZeroTemplateConsts.MultiTenancyEnabled ? null : (int?)1;
        return new List<ApplicationLanguage>
            {
                new ApplicationLanguage(tenantId, "en", "English", "famfamfam-flags us"),
                new ApplicationLanguage(tenantId, "pl", "Polski", "famfamfam-flags pl"),
                //Other languages
            };
    }

    //...
}
```

In the code above:

- "pl" is the language code for Polish.
- "Polski" is the display name.
- "famfamfam-flags pl" refers to the flag icon for Polish.

#### Create the Localization XML File

The localization XML file contains key-value pairs for translating text within the application. Each language has its own XML file where you define translations for UI elements, messages, and labels. 
This file ensures that the application can display content in the selected language. Create a new localization file in the **Localization** folder of the `YourProjectName.Core` project.

**YourProjectName-pl.xml**
```xml
<?xml version="1.0" encoding="utf-8" ?>
<localizationDictionary culture="pl">
  <texts>
    <text name="HomePage">Strona główna</text>
    <text name="Logout">Wyloguj się</text>
    <!-- Add other translations here -->
  </texts>
</localizationDictionary>
```

### 2. Add Language to Angular

To fully support the new language in the Angular UI, you need to update various localization settings, including date picker localization.

#### Configure Date Localization in ngx-bootstrap

To ensure that the ngx-bootstrap date picker supports the new language, update the `ngx-bootstrap-datepicker-config.service.ts` file. This configuration allows the date picker to display dates in the correct format for the selected language.

**Import the Locale**

Add the new language locale to the imports from `ngx-bootstrap/chronos`:

```javascript
import {
    enGbLocale,
    plLocale, // Add import for the new language here
    //Other locales
} from 'ngx-bootstrap/chronos';
```

**Update the `registerNgxBootstrapDatePickerLocales` Method**

Modify the method to include the new language inside the locales object:

```javascript
static registerNgxBootstrapDatePickerLocales(): Promise<boolean> {
    if (abp.localization.currentLanguage.name === 'en') {
        return Promise.resolve(true);
    }

    const locales: { [key: string]: any } = {
        'en': enGbLocale,
        'pl': plLocale, // Add new language here
        //Other languages
    };

    let localeKey = abp.localization.currentLanguage.name.toLowerCase();

    if (locales[localeKey]) {
        defineLocale(localeKey, locales[localeKey]);
        return Promise.resolve(true);
    }

    return Promise.reject(`Locale ${localeKey} not found`);
}
```

This method ensures that the correct locale is loaded based on the currently selected language in the application. If the language is not found in the locales object, an error is returned.

### 3. Verify and Test the Localization Changes
After implementing the necessary changes, follow these steps to verify that the new language is properly integrated into the application:

- **Restart the Application**
    - Restart both the backend and frontend to ensure that all localization changes take effect.
- **Select the New Language**
    - In your app, select the newly added language from the language selection drop-down menu and verify that user interface elements such as menus and labels are updated appropriately.
- **Test Date Localization**
    - Open any date pickers and confirm that dates are displayed in the correct format for the selected language.

> Note: The newly added language and its XML content are stored in the database when the application runs.

## Conclusion
By following these steps, you have successfully added support for a new language in your ASP.NET Zero Angular application. The localization process ensures that both backend and frontend components, including UI elements and date pickers, properly reflect the new language.

For more information, refer to the official documentation: 
- [ASP.NET Boilerplate Localization](https://aspnetboilerplate.com/Pages/Documents/Localization).
- [ASP.NET Zero Language Management](https://aspnetboilerplate.com/Pages/Documents/Zero/Language-Management).