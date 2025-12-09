**Title:** Introducing ASP.NET Zero v15
**Description:** ASP.NET Zero v15.0 brings .NET 10 and ABP 11.0 upgrades, improved multi tenancy, enhanced login & email flows, UI/UX refinements, updated Power Tools, and various performance and stability fixes.

# Introducing ASP.NET Zero v15.0

We're excited to announce the release of ASP.NET Zero v15.0, an update that includes package upgrades, enhanced multi-tenant features, improved user experience, and various enhancements and bug fixes. This release modernizes the core framework, improves stability, and continues our commitment to delivering a secure, scalable, and developer friendly application foundation.

## 🚀 Upgraded to .NET 10

ASP.NET Zero now fully supports .NET 10, leveraging improved performance, runtime stability, and the latest features of the .NET ecosystem. This upgrade ensures long-term compatibility and provides developers with an optimized backend environment.

Key benefits include:

* Faster runtime and improved memory usage
* Updated C# language features for clearer, more concise code
* Better tooling and diagnostics

## 🚀 Upgraded to ABP 11.0

We have upgraded ASP.NET Zero to run on **ABP 11.0**. Since AutoMapper moved to a commercial license model, a new package for Mapperly named `Abp.Mapperly` is added.

### 🔄 Transition to Mapperly

With **ABP 11.0**, the platform has transitioned to **Mapperly** as the default mapping approach, replacing many AutoMapper usages.

This transition provides:

* Faster compile time generated mappings
* Reduced runtime overhead
* More predictable and maintainable mapping code

This change results in a simpler and more efficient mapping structure across the application.

> **Note:** With this issue (see https://github.com/aspnetzero/aspnet-zero-core/issues/5838
), ASP.NET Zero is planned to adopt Mapperly (together with ABP 11.0) as its default object mapping approach.

## 📦 Private NuGet Packages for ASP.NET Boilerplate

Starting with **ASP.NET Zero v15.0**, ASP.NET Boilerplate packages are now distributed as **private NuGet packages** instead of open-source package references.

> **Note:** With ABP 11, open source development for ASP.NET Boilerplate packages will only contain bugfixes and we will not update open-source ABP to .NET 10. We will however continue development privately, and updates will continue to be delivered through private NuGet packages for ASP.NET Zero customers.


## 🔗 Improved Multi Tenancy Support for Account Link Modal

In v15.0, we enhanced the existing multi tenancy support in Account Link mode and made it more robust and reliable.

These improvements ensure that:

* Users can link accounts more consistently across tenants
* Linked account transactions behave more predictably in multi tenancy environments
* Security and authorization flows better prevent cross tenant data leakage

This improvement provides greater stability and flexibility for enterprise applications with complex tenant structures.

## 🎨 Improved Email Activation & Login Experience

Version 15.0 includes several updates to improve the user onboarding and authentication flow:

* **Refactored email confirmation logic** for more reliable email activation
* **Caps Lock indicator improvements**, including better behavior in Safari and MVC login screens
* **Streamlined password input UX**, including improved show/hide password support

These improvements create a more seamless authentication experience for end users.

## 🛠️ Enhancements & Fixes Across the Framework

This release includes many improvements across the Angular, MVC, and backend layers. A few notable examples:

### General Improvements

* More consistent **date/time pickers** with added disabled state support
* Improved responsiveness of the **Linked Accounts** menu on mobile devices
* Enhanced rendering and navigation of **external menu items**
* Fixes for **2FA remember client token** duration handling

### Power Tools Improvements

* Better handling of entity generation scenarios
* Full support for **relative nested namespaces**
* Fixes for entity naming edge cases

### UI/UX Fixes

* Corrected layouts and interactions in several components
* Fixed Quick Nav search rendering
* More stable behavior in charts, dashboards, and modal dialogs

All these updates together contribute to a more polished, reliable, and developer-friendly platform.

## 🗑️ Health Checks UI Temporarily Removed

Starting from v15.0, we have **removed the Health Checks UI dependency** due to stability issues and outdated packages.

However, this is **temporary**.

We plan to reintroduce a modernized and improved Health Checks UI in a future release with:

* Updated libraries
* Better customization options
* More resilient monitoring capabilities

Stay tuned for the upcoming version.

## ✅ Other Notable Improvements

* Updated NPM dependencies for the frontend frameworks
* Improvements in SignalR reconnect behavior
* Fixes for logout/session invalidation issues on Firefox
* Refactored email activation flow to improve long-term maintainability

## 🔗 Additional Resources

📢 Full changelog for v15.0 is available here:
[https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.0.0](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.0.0)

📢 v15.0 RC-1 Release Notes:
[https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.0.0-rc.1](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.0.0-rc.1)


## 🙏 **Conclusion**

ASP.NET Zero v15.0 modernizes the platform with upgrades to .NET 10 and ABP 11.0, while also incorporating numerous enhancements that improve usability, maintainability, and multi tenant functionality.

We hope this release improves your development experience and look forward to your feedback.

For questions, suggestions, or support, feel free to reach us at:
📩 **[info@aspnetzero.com](mailto:info@aspnetzero.com)**