### Overall

While base functionalities are identical for all versions of ASP.NET
Zero, there are still some differences between different versions. In
this document, we will highlight these differences.

#### ASP.NET Core v.s. ASP.NET MVC 5.x

Beginning from [v4.1](Change-Logs.md), we are more focused to
**ASP.NET Core** based solutions (rather than ASP.NET MVC 5.x) since
ASP.NET Core is Microsoft's new ASP.NET version. That means our new
major features will be implemented for ASP.NET Core version (.NET Core &
full .net framework).

#### Angular v.s. AngularJS 1.x

Beginning from [v4.1](Change-Logs.md), we are more focused to
**Angular** based solutions (rather than AngularJS 1.x) since Angular is
Google's new SPA framework. That means our new major features will be
implemented for Angular framework.

### Version Differences Table

The table below shows **only the differences, not all features**.

| Feature                                  | ASP.NET Core & Angular  | ASP.NET Core & jQuery   | ASP.NET MVC 5.x & AngularJS | ASP.NET MVC 5.x & jQuery |
| ---------------------------------------- | ----------------------- | ----------------------- | --------------------------- | ------------------------ |
| User Interface (theme)                   | Metronic v5.x           | Metronic v5.x           | Metronic v4.6.x             | Metronic v4.6.x          |
| ORM                                      | EF Core                 | EF Core                 | EF 6.x                      | EF 6.x                   |
| Xamarin Application                      | OK                      | OK                      | -                           | -                        |
| Tenant Subscription Management & Paypal Integration | OK                      | OK                      | -                           | -                        |
| Host Statistics Dashboard                | OK                      | OK                      | -                           | -                        |
| Identity Server Integration              | OK                      | OK                      | -                           | -                        |
| Chat & Real Time Notifications (SignalR integration) | OK (for .net framework) | OK (for .net framework) | OK                          | OK                       |
| ADFS login                               | -                       | -                       | OK                          | OK                       |
| Social Logins (Facebook, Twitter, Microsoft, Google+) | OK (Facebook, Twitter)  | OK (all)                | OK (all)                    | OK (all)                 |
| LDAP login                               | OK (for .net framework) | OK (for .net framework) | OK                          | OK                       |
| Two Factor Auth                          | SMS, Email, Google Auth | SMS, Email, Google Auth | SMS, Email                  | SMS, Email               |
| Application Setup Screen                 | OK                      | OK                      | -                           | -                        |


See **development guide** documents for details of all features in
different versions:

-   [ASP.NET Core & jQuery](Development-Guide-Core.md)
-   [ASP.NET Core & Angular](Development-Guide-Angular.md)
-   [ASP.NET MVC 5.x & jQuery](Development-Guide-Mvc-Angularjs.md)
-   [ASP.NET MVC 5.x & AngularJS 1.x](Development-Guide-Mvc-Angularjs.md)
