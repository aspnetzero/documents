**Title:** Introducing ASP.NET Zero v14.2
**Description:** ASP.NET Zero v14.2 is here, focusing on security enhancements like single refresh tokens, significant package upgrades including ABP 10.2 and Stripe v48.1, and numerous UI/UX improvements for a more robust and streamlined development workflow.

# Introducing ASP.NET Zero v14.2

We are pleased to announce the release of ASP.NET Zero v14.2. This version includes many improvements, major package upgrades, and bug fixes to make your development process more secure, efficient, and reliable. This update brings important security improvements by enforcing the use of single refresh tokens, fully upgraded dependencies, enhanced file validation, and many UI/UX improvements.

This update brings many important improvements, such as enforcing the use of single refresh tokens, fully upgraded front-end to Angular v20, the latest ABP 10.2 framework, and enhancements to simplify workflows.

Explore the main highlights of the v14.2 version.

## 🔒 Enhanced Security with Single Refresh Token

To make the application more secure against modern threats, we redesigned the authentication controller and now enforces a valid refresh token per user. This important update ensures that each user can only use one valid refresh token at a time. All previous tokens will automatically expire after re-login. This measure significantly reduces the risk of tokens being abused by infected devices and improves the overall security of ASP.NET Core applications. In addition, this approach makes session management more efficient and prevents stale tokens from accumulating in the cache.

## 🚀 Upgrade Angular v20

We modernized our entire frontend stack by upgrading to Angular v20, providing new features that improve performance, responsiveness, and the overall developer experience. We integrated the latest patterns and APIs, including:

- **New Control Flow Syntax:** the new built-in control flow (@if, @for, @switch) has replaced our templates. As a result, many components no longer require CommonModule, leading to more readable, type-safe templates and notable performance improvements.
- **Features Based on Signals:** This improvement is based on the new reactivity model. Now, we use leverage:
  - **Signal Inputs and Outputs:** For more effective and responsive component communication, use the new input() and output() functions.
  - **Signal Based inquiries:** The viewChild and contentChild inquiries can now be used as signals, which makes it easier to notice changes and improves the predictability of component interactions.
- **Inject() for Flexible Dependency Injection:** Nowadays, the inject() function offers a more versatile and potent method of managing dependency injection outside of conventional class constructors, as in utility functions and route guards.
- **Lazy Loaded Routes Optimized:** To fully benefit from the most recent advancements in lazy loading, we have improved our routing setup. This ensures that modules are loaded when needed for a quicker initial application startup.

## 🚀 Upgraded Core Dependencies: ABP 10.2 & Stripe v48.1

Keeping up with the ecosystem is of utmost importance. This release includes:

- **ABP 10.2: Upgraded:** The most recent version of the ASP.NET Boilerplate framework is now supported by ASP.NET Zero, giving you access to all of the new features and performance advantages of ABP v10.2.
- **Upgraded Stripe Package to v48.1.0:** We have updated our Stripe integration to the latest version, addressing breaking changes related to invoice payment handling. This ensures your payment processing remains seamless and secure.

## 📄 Advanced File Validation

To enhance data integrity at the upload point, we have added new file validators for popular document types. By adding server-side validation for Word, Excel, and PDF documents, this feature lowers the possibility of errors in business operations by guaranteeing that users can only submit files that adhere to the necessary standards and structure.

## 🛠️ Power Tools & MAUI Enhancements

Much attention has been paid to our Power Tools in an effort to enhance your code generation experience. The inclusion of the **Entity Detail View Page to the Power Tool for MAUI**, which makes it easier to create detailed views in your mobile applications, is a major feature for mobile developers. Additionally, we fixed a number of issues with entity history, navigation properties, and Excel imports, making the Power Tools more stable and dependable than before.

## 🎨 UI/UX Refinements

With a number of UI/UX improvements, this edition offers a more polished and uniform user experience throughout the application:

- **Redesigned Install Page:** To make the setup and installation page more contemporary and user-friendly, it has been aesthetically modified.
- **Improved Form Validation:** To make data entering easier and more intuitive, we have improved keypress handling and form validation logic.
- Dashboard Widgets That Respond: Dashboard widgets now adapt to mobile devices automatically, giving customers a smooth experience on all screen sizes.
- **MVC UI Consistency:** For a more cohesive appearance and feel, we've matched the Angular UI style with a number of MVC components, including the New Tenant page.

## ✅ Other Notable Improvements

- **Upgraded NPM & Swashbuckle Packages:** The most recent stable versions of every NPM package have been installed. In order to improve the capabilities of API documentation, we have also updated `Swashbuckle.AspNetCore` to version 8.1.2.
- **C# Codebase Updated:** By moving the remaining block scoped namespaces to file scoped namespaces and reworking to employ async lambdas, we are further modernizing our codebase, making it easier to read and more compliant with the most recent C# standards.
- **All-inclusive Bug Repairs:** The user delegation modal, permission filter persistence, logo uploads in different settings, and payment-related pages are just a few of the many bugs we have fixed throughout the framework.

## 🔗 Additional Resources

📢 Check out the full changelog for a detailed list of all enhancements and fixes:

[ASP.NET Zero v14.2 Release Notes](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v14.2.0)

[ASP.NET Zero v14.2 RC-1 Release Notes](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v14.2.0-rc.1)

## ✏️ Don't Miss Out!

We’ve published new blog posts to help you make the most of ASP.NET Zero:

📌 [Introducing ASP.NET Zero v14.1](https://aspnetzero.com/blog/introducing-asp.net-zero-v14.1)

📌 [How to Implement Dynamic Permissions in ASP.NET Zero](https://aspnetzero.com/blog/how-to-implement-dynamic-permissions-in-asp.net-zero)

📌 [Use API Versioning in ASP.NET Zero](https://aspnetzero.com/blog/use-api-versioning-in-asp.net-zero)

Check them out and stay ahead with the latest best practices and features!

## 🙏 Conclusion

With ASP.NET Zero v14.2, we’ve taken things even further. We hope you enjoy the latest improvements and look forward to your feedback. Feel free to reach out if you have any questions or comments.

<a href="mailto:info@aspnetzero.com">info@aspnetzero.com</a>