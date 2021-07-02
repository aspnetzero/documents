# Adding a New Menu Item

Let's begin from UI and create a new page named "**Phone book**".

## Defining a menu item

**AppNavigationProvider** class defines menus in the application. When
we change this class, menus are automatically changed. 

**AppNavigationProvider** class can be found in **.Web.Mvc** under APP (Or Application Area Name ) > Startup 

Open this class and create new menu item as shown below (You can add it right after the
dashboard menu item).

```csharp
.AddItem(new MenuItemDefinition(
    AppPageNames.Tenant.PhoneBook,
    L("PhoneBook"),
    url: "App/PhoneBook",
    icon: "glyphicon glyphicon-book"
    )
)
```

Every menu item must have a **unique name** to identify this menu item.
Menu names are defined in AppPageNames class as constants. We add a new
constant: "**PhoneBook**".

**AppPageNames** class can be found in **.Web.Mvc** under APP (Or Application Area Name ) > Startup 



## Localizing Menu Item Display Name

A menu item should also have a **localizable shown name**. It's used to
display menu item on the page. **L("PhoneBook")** is the localized name of
our new menu. **L** method is a helper method gets a localization key
and simply returns a **LocalizableString** object (see
AppNavigationProvider class).

Localization strings are defined in **XML** files under Localization folder in **.Core** project
as shown below:

<img src="images/localization-files-5.png" alt="Localization files" class="img-thumbnail" />

Open PhoneBook.xml (the **default**, **English** localization
dictionary) and add the following line:

```xml
<text name="PhoneBook">Phone book</text>
```

If we don't define "PhoneBook"s value for other localization
dictionaries, default value is shown in all languages. We can define it
also for Turkish in PhoneBook-tr.xml file:

```xml
<text name="PhoneBook">Telefon Rehberi</text>
```

## Other menu item properties

**url** can be a URL (it's URL of an **MVC Action** here) that will be
redirected when we click the menu item.

Lastly, **icon** is the shown menu icon for new menu item. It can be a
**css** class. We can use Glyphicon, Font-Awesome or another css font
library here.

See [navigation
document](https://aspnetboilerplate.com/Pages/Documents/Navigation) for
more information on menu definitions.

## Next

- [Creating the Page](Developing-Step-By-Step-Core-Creating-Page.md)
