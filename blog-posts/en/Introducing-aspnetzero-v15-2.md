**Title:** Introducing ASP.NET Zero v15.2
**Description:** ASP.NET Zero v15.2 brings a brand new React UI with Power Tools support, Angular 21 upgrade, Active Session management, AI development skills, the return of Health Checks UI, and various performance improvements.

# Introducing ASP.NET Zero v15.2

We are thrilled to announce the release of **ASP.NET Zero v15.2**! This release marks a significant milestone for the framework, introducing a highly anticipated frontend option, major framework upgrades, and powerful new features designed to enhance security, developer productivity, and user experience.

Here is a detailed look at what is new in v15.2.

## 🚀 Upgraded to ABP 11.1 & Mapperly Migration

ASP.NET Zero v15.2 upgrades the core backend infrastructure to **ABP v11.1**.

Building on the foundation laid in v15.0, we have also finalized the **migration from AutoMapper to Mapperly**. Mapperly is now deeply integrated, including inside the ASP.NET Zero Power Tools. This means all your generated code will now utilize Mapperly's fast, compile time mapping generation, drastically reducing runtime overhead.

## 🚀 Upgraded to Angular 21 & Zoneless Change Detection

For our Angular developers, we have made a massive leap by upgrading the Angular app directly to **Angular v21**.

Along with the version bump and updating the related NPM packages, we have removed `Zone.js` to enable **Zoneless Change Detection**. This provides a significant boost to frontend performance, reduces bundle sizes, and modernizes the application lifecycle. Additionally, we have migrated to new routing configurations and removed deprecated modules.

## ⚛️ React UI Enhancements

Following the initial release of the **React UI** in v15.1, we have been working hard to expand its capabilities and bring even more features to our modern, fast frontend.

In v15.2, we have added several major features to the React UI:

- **Social Login:** Seamlessly integrate third party authentication providers into your React app.
- **QR Code Login:** Added QR code login support powered by SignalR integration for a faster authentication flow.
- **Dynamic Entity Property Management:** This powerful management feature is now fully supported in the React frontend.
- **Customizable Dashboards:** Introduced a new drag and drop functionality, allowing users to easily customize their dashboard layouts.
- **Excel Export Selection:** Added a new modal allowing users to select specific Excel columns before exporting data.

You can find more information about [React UI here](https://aspnetzero.com/react).

## ⚡ ASP.NET Zero Power Tools Support for React

To make adopting the new React frontend as seamless as possible, **ASP.NET Zero Power Tools now fully supports the React UI!** This Rapid Application Development capability means you can generate your entities, endpoints, and React user interfaces automatically, just like you already do with Angular and MVC. This will significantly speed up your development process and help you build React applications efficiently.

## 🔐 Active Session Management & Revocation

Security and user control are paramount. In v15.2, we are introducing the **Active Session** feature.

This new implementation allows users to monitor their active sessions across different devices and browsers. It includes a complete UI redesign for session management and introduces **conditional session revocation**. If an account is compromised or a user simply forgot to log out from a public device, they can now easily view and revoke those active sessions directly from their security settings, preventing unauthorized access and session hijacking.

![Active Session Page](/Images/Blog/active-session-page.png)

You can find detailed instructions on how to use the Active Session feature in [this document](https://docs.aspnetzero.com/aspnet-core-angular/latest/Features-Angular-Active-Sessions).

## 🤖 AI Development Skills & Guidelines

As AI-assisted coding becomes the standard, we want ASP.NET Zero developers to get the best possible results from their AI tools.

We have introduced **AI Development Rules & Guidelines** directly into the project. Specifically, we added AI skills and context files tailored for **Claude** and **Cursor**. These guidelines help your AI assistants understand ASP.NET Zero's architecture, naming conventions, and best practices, making AI driven development smoother and more accurate.

## 🩺 Health Checks UI is Back!

In v15.0, we temporarily removed the Health Checks UI dependency due to stability issues with outdated third-party packages. We are happy to announce that **Health Checks UI has been completely re implemented and restored** in v15.2, providing you with a clean, visual dashboard to monitor the health and status of your application's databases, APIs, and microservices.

![Health Checks Page](/Images/Blog/health-checks-page.png)

## 🎨 UI/UX & General Enhancements

This release is packed with quality of life improvements, UI refinements, and bug fixes:

### Framework & Tooling Improvements

- **Swashbuckle Upgrade:** Swagger UI has been upgraded to Swashbuckle v10.x.
- **Power Tools on ARM:** Added robust support for the Power Tools GUI on ARM based platforms (like Apple Silicon MacBooks).

### UI & Styling Fixes

- **View Notification Modal:** Added a dedicated message detail view modal to the Notifications page for both Angular and MVC.
- **Cross Browser Consistency:** Fixed Safari sidebar background alignments and resolved vertical scrolling issues on macOS.
- **Refactored Components:** Updated file upload components to use PrimeNG and converted `PrimengTableHelper` to use modern Angular signal structures.

## 🔗 Additional Resources

Full changelog and commit history for v15.2 are available here:

[https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.2.0](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.2.0)

📢 v15.2 RC-1 Release Notes:
[https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.2.0-rc.1](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.2.0-rc.1)

## 🙏 Conclusion

ASP.NET Zero v15.2 is a massive leap forward. Whether you are excited to build your next project with the new **React UI**, eager to utilize the blistering speed of **Angular 21**, or looking to secure your application with **Active Sessions**, this release has something for everyone.

We hope these new features and upgrades significantly improve your development experience.

For questions, suggestions, or support, feel free to reach us at:
📩 **[info@aspnetzero.com](mailto:info@aspnetzero.com)**
