**Title:** Introducing ASP.NET Zero v15.4
**Description:** ASP.NET Zero v15.4 upgrades ABP packages to 11.3.0, adds customizable email templates, introduces Power Tools Global .NET Tool with Web UI and Regenerate All, migrates frontend tooling from Yarn to pnpm, improves OpenID Connect authorization code flow security, publishes Elsa v3 integration resources, and brings important security and UI fixes.

# Introducing ASP.NET Zero v15.4

We are excited to announce the release of **ASP.NET Zero v15.4**! This release brings a solid set of framework upgrades, new productivity features, package updates, security improvements, and UI fixes based on community feedback.

Here is a detailed look at what is new in v15.4.

## 🚀 Upgraded to ABP 11.3.0

ASP.NET Zero v15.4 upgrades the underlying ASP.NET Boilerplate packages to **ABP 11.3.0**.

This update includes several important framework-level improvements:

- **Redis configuration improvements:** Added Redis `ConfigurationOptions` customization hooks, making it easier to support advanced Redis scenarios such as Azure managed identity authentication with Microsoft Azure StackExchangeRedis.
- **Mapperly projection support:** Improved `ObjectMapper.ProjectTo` support for Mapperly projection mappers and added clearer guidance when a projection mapper is missing.
- **EF Core cascade soft delete:** Added opt-in cascade soft delete support for EF Core relationships.
- **EF Core update optimization:** Added opt-in support for skipping updates when entities are unchanged, together with fixes for edge cases around update detection.
- **Dynamic sorting hardening:** Restricted dynamic sorting selectors to property and field access to avoid unsafe or unsupported sorting expressions.
- **Soft-delete query fixes:** Improved soft-delete DbFunction handling and added regression coverage for literal mapping scenarios.

We also updated related dependencies and cleaned up package references, including `SourceLink.GitHub` and `ConfigureAwait.Fody` updates.

## 📧 Customizable Default Email Templates

One of the main features of v15.4 is the new **Email Templates** management screen.

Host administrators can now customize the default email templates directly from the application. This makes it much easier to adapt system emails to your brand without manually changing embedded HTML resources.

The new feature allows you to:

- View the built-in application email templates
- Edit template HTML with a code editor
- Preview the email before saving
- Enable or disable a customized template
- Reset a customized template back to its default version

This feature covers common application emails such as account activation, password reset, passwordless login, password changed, email change, chat notification, and subscription notification emails.

![Email Templates Page](/Images/Blog/email-templates-page.png)

## ⚡ Power Tools Global .NET Tool with Web UI

ASP.NET Zero Power Tools now has a new **Global .NET Tool** option with a **Web UI**.

This gives developers a modern, cross-platform way to use Power Tools without depending only on the Visual Studio extension. It is especially useful for teams working on macOS, Linux, CI environments, or mixed development setups.

With the new Power Tools experience, you can also use the **Regenerate All** feature to regenerate all Power Tools definitions in a single flow instead of regenerating entities one by one. This is helpful after upgrading ASP.NET Zero, updating Power Tools templates, or applying project-wide code generation changes.

You can read the dedicated blog post here:

[Global .NET Tool for Power Tools](https://aspnetzero.com/global-dotnet-tool-for-power-tools)

## 📦 Migrated from Yarn to pnpm

In v15.4, we migrated frontend package management from **Yarn v1/classic** to **pnpm**.

Yarn classic is deprecated and `yarn audit` is no longer reliable for modern projects. With this change, ASP.NET Zero now uses `pnpm@9.15.4` across the Angular app, React app, MVC/Public web projects, Web.Host frontend resources, and Playwright UI tests.

This provides:

- Faster and more deterministic dependency installation
- Better monorepo-friendly package management behavior
- A modern replacement for deprecated Yarn classic workflows
- More consistent frontend tooling across ASP.NET Zero UI projects

## 🔄 Package Updates

This release includes package upgrades across the backend and frontend stacks.

Highlights include:

- Upgraded NuGet packages, including ABP package upgrades
- Updated NPM packages
- Updated `abp-web-resources`
- Migrated package manager scripts and lock files to pnpm
- General dependency cleanup for a more maintainable solution

These updates help keep ASP.NET Zero projects aligned with current tooling and dependency versions.

## 🧩 Elsa v3 Integration Resources

We also published new resources for integrating **Elsa v3** with ASP.NET Zero.

Elsa v3 is a powerful workflow engine for .NET applications. The new integration content explains how to use Elsa with ASP.NET Zero, including backend setup, authentication, multi-tenancy, and UI integration points.

You can find the related posts here:

- [How to Integrate Elsa v3 with ASP.NET Zero - Part 1: Backend Setup](https://aspnetzero.com/how-to-integrate-elsa-v3-with-aspnet-zero-part-1)
- [How to Integrate Elsa v3 with ASP.NET Zero - Part 2: Angular Frontend Integration](https://aspnetzero.com/how-to-integrate-elsa-v3-with-aspnet-zero-part-2)

We also published guidance for the ASP.NET Core MVC & jQuery architecture to help MVC users integrate Elsa 3.x into their projects.

## 🔑 OpenID Connect Authorization Code Flow Improvements

v15.4 includes an important security improvement for **OpenID Connect authorization code flow**.

Previously, the Angular login callback used the frontend OpenID Connect library flow in a way that could load the discovery document and attempt to exchange the authorization code from the browser. That behavior is acceptable for frontend-only applications, but ASP.NET Zero applications have a backend and can use a more secure confidential-client style flow.

With this update, when OpenID Connect is configured with `ResponseType` set to `code`, the Angular app no longer calls `loadDiscoveryDocumentAndTryLogin()` in the callback path. Instead, it sends the authorization `code`, PKCE verifier, redirect URI, and return URL to the backend. The backend then exchanges the code with the identity provider and uses the configured OpenID Connect settings, including `ClientId`, `ClientSecret`, `Authority`, `ValidateIssuer`, and claims mapping.

This improves security by keeping token endpoint communication on the server side and avoiding access token exposure in the browser. It also works better with identity providers such as Azure AD where a Web application registration requires a client secret at the token endpoint.

## 🔐 Security Improvements

v15.4 includes several important security hardening improvements.

Notable fixes include:

- Hardened `GetProfilePictureByUser` so profile pictures cannot be retrieved across tenants without proper access checks.
- Improved OpenID Connect authorization code flow handling by moving the code exchange to the backend.
- Addressed security issues found during white-box testing.
- Added framework-level dynamic sorting hardening through ABP 11.3.0.

These changes improve tenant isolation, authentication flow safety, and backend query safety.

## 🅰️ Angular UI Fixes

This release also includes several Angular UI fixes and refinements.

Highlights include:

- Restored missing `AccountRouteGuard` protection for account routes.
- Fixed continuous API calls and reloading in Organization Unit members and roles grids.
- Fixed permission tree search filtering behavior with PrimeNG 21.
- Improved stacked alert behavior.
- Continued package and tooling updates after the Angular 21 upgrade.

These fixes make the Angular application more stable and predictable in day-to-day usage.

## 🛠️ General Enhancements and Fixes

In addition to the major items above, v15.4 includes many smaller improvements and bug fixes across ASP.NET Zero.

Some notable examples:

- Improved role name handling and role management behavior.
- Improved `ObjectMapper.ProjectTo` behavior after the Mapperly migration.
- Updated cache documentation for Redis configuration.
- Fixed EF Core skip-update change detection edge cases.
- Improved generated and framework package references.
- Applied general UI, backend, and tooling fixes based on reported issues.

## 🔗 Additional Resources

Full changelog and commit history for v15.4 are available here:

[https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.4.0](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.4.0)

📢 v15.4 RC-1 Release Notes:
[https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.4.0-rc.1](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v15.4.0-rc.1)

## 🙏 Conclusion

ASP.NET Zero v15.4 is a practical release that improves the platform in multiple areas: ABP 11.3.0, customizable email templates, new Power Tools workflows, pnpm-based frontend tooling, OpenID Connect authorization code flow hardening, Elsa v3 integration resources, package upgrades, security hardening, and UI stability fixes.

We hope these improvements make your development workflow smoother and your applications easier to maintain.

For questions, suggestions, or support, feel free to reach us at:
📩 **[info@aspnetzero.com](mailto:info@aspnetzero.com)**
