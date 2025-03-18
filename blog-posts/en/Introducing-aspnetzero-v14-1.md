**Title:** Introducing ASP.NET Zero v14.1
**Description:**  ASP.NET Zero v14.1: Angular standalone components, MAUI QR login, package updates, and bug fixes for a more efficient development experience.

# Introducing ASP.NET Zero v14.1

We are thrilled to announce ASP.NET Zero v14.1, packed with new features, enhancements, and important fixes to improve your development experience! This release focuses on Angular standalone components, QR login for MAUI apps, package upgrades, permission search enhancements, and various bug fixes to make your applications faster, more secure, and more efficient.

Let’s dive into the key highlights of this release!

## 📌 Standalone Components in Angular
With this release, ASP.NET Zero now leverages Angular Standalone Components to simplify Angular application development. This enhancement removes the need for NgModules, making components more modular, easier to reuse, and improving overall performance. This shift allows developers to create truly independent components, streamlining the development process and reducing boilerplate code. Furthermore, standalone components facilitate better tree-shaking, resulting in smaller bundle sizes and faster load times.

## 📲 QR Login from MAUI Application
You can now enable QR-based authentication in your MAUI apps! This feature allows users to scan a QR code to log in quickly and securely without manually entering their credentials. Using your MAUI app, you can scan a QR code displayed in a web app and seamlessly log in to that web app with the same user credentials generated on your MAUI device. This development aims to streamline the user experience and bridge the gap between mobile and web authentication, providing a modern and efficient login alternative.

When QR Login is enabled from the Settings page, a QR code will be displayed on the Login page as shown below. With this activation, you will also see that the QR Login scanner menu item in the sidebar menu of the MAUI app is activated.

![QR Login Page](/Images/Blog/qr-login.png)

> This image shows the login page of a Web app with a displayed QR code, which users can scan to log in securely without entering their credentials manually.

![QR Menu Item MAUI](/Images/Blog/qr-menu-item-maui.jpg)

> This image shows the QR Login scanner menu item in the sidebar menu of the MAUI app, which gets activated once QR Login is enabled in the app's settings.

## Angular Application Build System Migration
Experience faster build times and optimized performance with the migration to Angular's latest build system in ASP.NET Zero v14.1. This update ensures your Angular applications leverage the most efficient compilation and bundling processes, resulting in quicker development cycles and improved application load times. We have also updated Karma to the common js format, and have removed deprecated async functions.This transition introduces ESM with dynamic imports, faster builds through esbuild and Vite, integrated SSR/prerendering, and automatic stylesheet hot replacement, enhancing your development workflow.

## 🔄 Upgraded NPM Packages
We've updated all possible NPM packages to their latest stable versions to ensure better performance, security, and compatibility with modern frontend development practices.

## 🚀 Upgraded to ABP 10.1
ASP.NET Zero now supports ABP 10.1, bringing new features and performance improvements that align with the latest .NET 9 capabilities.

For more information, see the [ABP 10.1 release notes](https://github.com/aspnetboilerplate/aspnetboilerplate/releases/tag/v10.1).

## 🔒 Upgrade OpenIddict 6.0
ASP.NET Zero v14.1 includes an upgrade to OpenIddict 6.0, enhancing the security and reliability of our authentication services. This update ensures improved compliance with the latest OpenID Connect and OAuth 2.0 standards, while also providing performance improvements.

## 🎨 PrimeNG Upgrade (v19)
We've upgraded PrimeNG to version 19, bringing UI enhancements and bug fixes to improve user experience. This update includes improvements to component styling, performance optimizations, and resolutions to known issues, resulting in a more polished and responsive user interface for your applications.

## 🔍 Improved Permission Search
When searching for permissions, sub-permissions will now be selectable along with main permissions, making it easier to manage complex permission structures. This enhancement simplifies the process of assigning and managing permissions, particularly in applications with intricate role-based access control requirements, giving administrators greater control.

## 🔑 Impersonator User Name in Change Logs
To enhance audit logs, the impersonated user's name is now displayed in the change logs table, improving tracking and transparency. This provides a more complete audit trail, allowing administrators to easily identify who performed actions while impersonating another user

## 🔄 Replaced adal-angular with msal-angular
We've fully transitioned from adal-angular to msal-angular, ensuring compatibility with modern authentication standards. This migration aligns with Microsoft's recommended authentication libraries, providing improved security, and simplifying integration with Microsoft Entra ID and other modern identity providers.

## 🔗 Additional Resources
📢 Check out the full changelog:

[ASP.NET Zero v14.1 Release Notes](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v14.1.0-rc.1)

## ✏️ Don't Miss Out!
We’ve published new blog posts to help you make the most of ASP.NET Zero:

📌 [How to Implement Dynamic Permissions in ASP.NET Zero](https://aspnetzero.com/blog/how-to-implement-dynamic-permissions-in-asp.net-zero)

📌 [Use API Versioning in ASP.NET Zero](https://aspnetzero.com/blog/use-api-versioning-in-asp.net-zero)

Check them out and stay ahead with the latest best practices and features!

## 🙏 Conclusion

With ASP.NET Zero v14, we've raised the bar yet again. We hope you enjoy this release and look forward to hearing your feedback. If you have any questions or comments, please don't hesitate to contact us. 

<a href="mailto:info@aspnetzero.com">info@aspnetzero.com</a>