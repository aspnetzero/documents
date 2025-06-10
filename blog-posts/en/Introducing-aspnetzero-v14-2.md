**Title:** Introducing ASP.NET Zero v14.2
**Description:** ASP.NET Zero v14.2 is here, focusing on security enhancements like single refresh tokens, significant package upgrades including ABP 10.2 and Stripe v48.1, and numerous UI/UX improvements for a more robust and streamlined development workflow.

# Introducing ASP.NET Zero v14.2

We are thrilled to announce ASP.NET Zero v14.2. This release is packed with significant enhancements, crucial package upgrades, and a host of bug fixes designed to make your development process more secure, efficient, and reliable. This update brings key improvements in security with single refresh token enforcement, major dependency upgrades, advanced file validation, and numerous UI/UX refinements.

Let's explore the key highlights of the v14.2 release!

## 🔒 Enhanced Security with Single Refresh Token

To bolster application security, we have refactored our authentication controller to enforce the use of a single valid refresh token per user. This crucial update ensures that each user can only have one active refresh token at a time, invalidating any previous tokens upon a new login. This measure significantly reduces the risk of token misuse from compromised devices and enhances the overall security posture of your application.

## 🚀 Upgraded Core Dependencies: ABP 10.2 & Stripe v48.1

Staying current with the ecosystem is a top priority. In this release, we have:

  * **Upgraded to ABP 10.2:** ASP.NET Zero now supports the latest version of the ASP.NET Boilerplate framework, bringing you all the performance benefits and new features from ABP v10.2.
  * **Upgraded Stripe Package to v48.1.0:** We have updated our Stripe integration to the latest version, addressing breaking changes related to invoice payment handling. This ensures your payment processing remains seamless and secure.

## 📄 Advanced File Validation

We've introduced new file validators for common document types to improve data integrity right at the point of upload. This feature adds server-side validation for Excel, PDF, and Word documents, ensuring that users can only upload files that meet the required format and standards, reducing potential errors in business workflows.

## 🛠️ Power Tools & MAUI Enhancements

Our Power Tools have received significant attention to improve your code generation experience. A key highlight for mobile developers is the addition of the **Entity Detail View Page to the Power Tool for MAUI**, simplifying the creation of detailed views in your mobile applications. We have also resolved several bugs related to Excel imports, entity history, and navigation properties, making the Power Tools more robust and reliable than ever.

## 🎨 UI/UX Refinements

This release brings a polished and more consistent user experience across the application with several UI/UX updates:

  * **Redesigned Install Page:** The setup and installation page has been visually updated for a more modern and user-friendly experience.
  * **Improved Form Validation:** We have enhanced form validation logic and keypress handling to provide a smoother and more intuitive data entry process.
  * **Responsive Dashboard Widgets:** Dashboard widgets now automatically resize for mobile devices, ensuring a seamless experience for users on any screen size.
  * **MVC UI Consistency:** We've aligned several MVC components, such as the New Tenant page, with the style of the Angular UI for a more unified look and feel.

## ✅ Other Notable Improvements

  * **Upgraded NPM & Swashbuckle Packages:** All possible NPM packages have been updated to their latest stable versions. We've also upgraded `Swashbuckle.AspNetCore` to v8.1.2 to enhance API documentation capabilities.
  * **Modernized C# Codebase:** We continue to modernize our codebase by migrating the remaining block-scoped namespaces to file-scoped namespaces and refactoring to use async lambdas, improving code readability and aligning with the latest C# standards.
  * **Comprehensive Bug Fixes:** We have squashed numerous bugs across the framework, including issues with the user delegation modal, permission filter persistence, logo uploads in various environments, and payment-related pages.

## 🔗 Additional Resources

📢 Check out the full changelog for a detailed list of all enhancements and fixes:

[ASP.NET Zero v14.2 Release Notes](https://www.google.com/search?q=https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v14.2.0)

## ✏️ Don't Miss Out!

We’ve published new blog posts to help you make the most of ASP.NET Zero:

📌 [Introducing ASP.NET Zero v14.1](https://aspnetzero.com/blog/introducing-asp.net-zero-v14.1)

📌 [How to Implement Dynamic Permissions in ASP.NET Zero](https://aspnetzero.com/blog/how-to-implement-dynamic-permissions-in-asp.net-zero)

📌 [Use API Versioning in ASP.NET Zero](https://aspnetzero.com/blog/use-api-versioning-in-asp.net-zero)

Check them out and stay ahead with the latest best practices and features!

## 🙏 Conclusion

With ASP.NET Zero v14.2, we’ve taken things even further. We hope you enjoy the latest improvements and look forward to your feedback. Feel free to reach out if you have any questions or comments.

<a href="mailto:info@aspnetzero.com">info@aspnetzero.com</a>