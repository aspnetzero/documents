# Localization

ASP.NET Zero **User Interface** is completely localized. ASP.NET Zero uses **dynamic, database based, per-tenant** localization (See the related section above).

XML files are used as base translation for desired languages:

<img src="images/localization-files-core-1.png" alt="Localization XML files" class="img-thumbnail" />

PhoneBook will be your ProjectName. You can add more XML files by copying one XML file and translate to desired language. See [valid culture codes](http://www.csharp-examples.net/culture-names/).

When you are adding a new localizable text, add it to the XML file of the default language then use in your application (Also, add translated values to corresponding XML files). No need to add it to database migration code since value in the XML file will be used as default.

**Application languages** are defined in **DefaultLanguagesCreator** class. This is used as a **seed data** in Entity Framework Migration. So, if you want to **add a new default language**, just add it into DefaultLanguagesCreator class. Also, you should add a corresponding XML file as described above as default translation.

See [localization](https://aspnetboilerplate.com/Pages/Documents/Localization) and [language management](https://aspnetboilerplate.com/Pages/Documents/Zero/Language-Management) documentations for more information.