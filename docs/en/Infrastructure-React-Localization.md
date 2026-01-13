# Localization

ASP.NET Zero **User Interface** is completely localized. ASP.NET Zero uses **dynamic, database based, per-tenant** localization.

XML files are used as base translations for desired languages (defined in the server-side); for defining and adding new localization, refer to the [How to Add a New Language in an ASP.NET Zero React Application](Adding-New-Localization-React).

<img src="images/localization-files-core-1.png" alt="Localization XML files" class="img-thumbnail" />

PhoneBook will be your ProjectName. You can add more XML files by copying one XML file and translate to desired language. See [valid culture codes](http://www.csharp-examples.net/culture-names/).

When you are adding a new localizable text, add it to the XML file of the default language then use in your application (Also, add translated
values to corresponding XML files). 

**Application languages** are defined in the **DefaultLanguagesCreator** class. This is used as seed data in the Entity Framework Migration. So, if you want to **add a new default language**, just add it into the **DefaultLanguagesCreator** class. This operation ensures that the new language is automatically seeded into the database during migration, making it available to the application at startup. This process occurs on the server side. You can refer to the documentation on [How to Add a New Language in an ASP.NET Zero React Application](Adding-New-Localization-React) for further guidance.

See [localization](https://aspnetboilerplate.com/Pages/Documents/Localization) and [language management](https://aspnetboilerplate.com/Pages/Documents/Zero/Language-Management) documentations for more information on localization.

