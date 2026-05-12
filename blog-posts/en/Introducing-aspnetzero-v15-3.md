**Title:** Introducing ASP.NET Zero v15.3
**Description:** ASP.NET Zero v15.3 brings an upgrade to ABP 11.2, Rate Limiting, External Login Linking, an Elsa 3.0 integration sample, comprehensive React UI tests, and various security and performance improvements.

# Introducing ASP.NET Zero v15.3

We are thrilled to announce the release of **ASP.NET Zero v15.3**. Building on the massive frontend updates of our previous releases, this version focuses on core framework upgrades, security features, workflow automation, and ensuring the absolute stability of our modern UIs.

Here is a detailed look at the highlights and what is new in v15.3.

## 🚀 Upgraded to ABP 11.2

ASP.NET Zero v15.3 upgrades the core backend infrastructure to **ABP v11.2**.

Keeping our underlying framework up-to-date ensures that your applications benefit from the latest performance optimizations, bug fixes, and core capabilities provided by the ABP framework, keeping your foundation rock solid and secure.

## 🚦 Rate Limiting & Throttling

To help you protect your applications from abuse and manage API traffic effectively, we have introduced a built-in **Rate Limiting and Throttling** feature.

This highly anticipated enhancement allows you to control the frequency of requests a user or client can make to your endpoints within a given timeframe. It provides a crucial layer of security to safeguard your backend resources against brute-force attacks, DDoS attempts, and unintentional traffic spikes.

![Rate Limiting Gif](/Images/Blog/rate-limiting.gif)

[Rate Limiting Document](https://docs.aspnetzero.com/aspnet-core-angular/latest/Features-Rate-Limiting)

## 🔗 External Login Linking

We have significantly improved authentication flexibility by adding **support for linking an existing membership account to a social login**.

Previously, users signing in via external providers (like Google, Microsoft, or Twitter) were treated as new users if they didn't already have a specifically linked account. Now, users who already have a local account can seamlessly link their preferred external/social login methods to their existing profiles, ensuring a unified and frictionless login experience.

![External Login Linking Gif](/Images/Blog/external-login-linking.gif)

[External Login Linking Document](https://docs.aspnetzero.com/aspnet-core-angular/latest/Features-Angular-Social-Account-Linking)

## ⚙️ Elsa 3.0 Integration Sample

For teams relying on workflow automation, we have updated our ecosystem by adding an **Elsa 3.0 integration sample**.

Elsa is a powerful workflow library that enables running native .NET workflows. By updating our Angular sample to Elsa 3.0, we are providing you with a modern, up-to-date reference architecture to easily integrate complex, visually designed workflows directly into your ASP.NET Zero applications.

[How to Integrate Elsa v3 with ASP.NET Zero - Part 1 Backend Setup](https://aspnetzero.com/blog/how-to-integrate-elsa-v3-with-aspnet-zero-part-1)

[How to Integrate Elsa v3 with ASP.NET Zero - Part 1 Angular Frontend Integration](https://aspnetzero.com/blog/how-to-integrate-elsa-v3-with-aspnet-zero-part-2)


## ⚛️ React UI Tests & Enhancements

Our modern **React UI** continues to mature, and in v15.3, we prioritized its stability and multi-tenant capabilities:

* **UI Tests:** We have created and implemented UI tests for the React frontend. This ensures ongoing stability, prevents regressions during future updates, and provides a reliable testing foundation for your own custom React code.
* **Subdomain-based Multi-Tenancy:** We have fully implemented subdomain-based multi-tenancy support for React, including the `PreventNotExistingTenantSubdomains` feature. Tenants can now have their own dedicated URLs (e.g., *tenant1.yourdomain.com*).

## 🛡️ Security & Session Improvements

Security and user flow consistency have received several important upgrades:

* **Password Reset Security:** We have enhanced the password reset flow's security and usability by introducing code expiration to prevent the reuse of old reset tokens.
* **Session Timeout:** Improved session timeout detection to work seamlessly across multiple inactive tabs and handle device lock scenarios more reliably.
* **OpenID Connect Return URL:** Preserved the `returnUrl` during the OpenID Connect login flow, ensuring users are correctly redirected back to their requested page after a successful login.
* **Dynamic Dashboard Permissions:** Introduced a new permission to strictly control dashboard edit modes, allowing administrators to dictate who can customize drag-and-drop layouts.

## 🛠️ Developer Experience & Strict Code Quality

v15.3 brings powerful new tooling and strictness constraints to the frontend to help teams write cleaner code:

* **Full TypeScript Strict Mode:** We have enabled full strict options for TypeScript and Angular, enforcing better type safety and helping catch potential bugs at compile time.
* **Prettier & Husky Pre-commit Checks:** Added a pre-commit hook integrating ESLint, Prettier, and Husky into the monorepo to ensure all committed code is automatically formatted and verified.
* **General Fixes:** Addressed mobile UI sidebar issues on iOS, fixed Recaptcha validation bugs in email activation flows, and resolved minor Mapperly runtime mapping regressions.

## 🔗 Additional Resources

Full changelog and commit history for v15.3 are available here:

📢 v15.3 Release Notes:
[https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.3.0](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.3.0)

📢 v15.3 RC-1 Release Notes:
[https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.3.0-rc.1](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.3.0-rc.1)

## 🙏 Conclusion

ASP.NET Zero v15.3 is all about robustness and control. Whether you are leveraging the new **Rate Limiting** to secure your APIs, linking accounts with our new **External Login flow**, or utilizing the updated **Elsa 3.0** workflows, we are confident this update provides immense value to your application's architecture.

We hope these new features and upgrades significantly improve your development experience.

For questions, suggestions, or support, feel free to reach us at:
📩 **[info@aspnetzero.com]()**