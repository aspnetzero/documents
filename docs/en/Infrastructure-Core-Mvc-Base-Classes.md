# Base Classes

There are some useful base classes used in the application:

- PhoneBook**AppServiceBase** can be used as a base class for all **application services**.
- PhoneBook**DomainServiceBase** can be used as a base class for **domain services**.
- PhoneBook**ControllerBase** can be used as a base class for ASP.NET Core **MVC Controllers**.
- PhoneBook**RazorPage** can be used as a base class for ASP.NET **MVC Views**. Actually, all views are automatically inherit this since it's defined in **\_ViewImports.cshtml** files. You can add some common properties/methods here to use in all views.
- PhoneBook**ServiceBase** can be used as a base class for other service-like classes. UserEmailer class inherits it, for instance.
- PhoneBook**RepositoryBase** can be used as a base class for [custom repository](https://aspnetboilerplate.com/Pages/Documents/EntityFramework-Integration#DocCustomRepositoryMethods) implementations.

It's strongly recommended to inherit one of these classes upon your needs since they really make Logging, Localization, Authorization... easier.