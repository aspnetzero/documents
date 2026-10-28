# Project Types Differences

## Overall

ASP.NET Zero is offered in three project types, all of them built on the same ASP.NET Core backend:

- **ASP.NET Core & jQuery** - a server rendered ASP.NET Core MVC application.
- **ASP.NET Core & Angular** - an Angular single page application talking to the `Web.Host` API.
- **ASP.NET Core & React** - a React single page application talking to the `Web.Host` API. Introduced in [v15.1](Change-Logs).

Because the backend is shared, everything that lives on the server side (application services, multi-tenancy, permissions, background jobs, webhooks, GraphQL, health check endpoints, the MAUI mobile app, the public web site, Power Tools code generation) behaves identically in all three. The differences below are all on the UI side.

### Legacy project types

**ASP.NET MVC 5.x** and **AngularJS 1.x** based project types are no longer developed. Beginning from [v4.1](Change-Logs), new features are implemented only for the ASP.NET Core based project types. See [Old Project Type Info](Old-Project-Type-Info) for details.

## Version Differences

### External login providers

| Provider | ASP.NET Core & jQuery | ASP.NET Core & Angular | ASP.NET Core & React |
|----------|:---------------------:|:----------------------:|:--------------------:|
| Facebook | Yes | Yes | Yes |
| Google | Yes | Yes | Yes |
| Microsoft | Yes | Yes | Yes |
| Twitter | Yes | Yes | Yes |
| OpenID Connect | Yes | Yes | Yes |
| WS-Federation | Yes | Yes | Yes |
| Apple | No | No | No |

> **Apple sign-in is supported by ASP.NET Zero, but in the [.NET MAUI mobile application](Development-Guide-MAUI) rather than in the web UIs.** "Sign in with Apple" is a native Apple-platform API, so it cannot be completed from a browser. The mobile app authenticates with `AppleSignInAuthenticator` and posts the resulting identity token to the API, which validates it through the `Authentication:Apple` section of `Web.Host`. The web UIs therefore leave Apple out on purpose: `Web.Mvc` does not register the provider at all, and the Angular and React login pages filter it out of the provider list returned by `GetExternalAuthenticationProviders`.

### UI libraries

| | ASP.NET Core & jQuery | ASP.NET Core & Angular | ASP.NET Core & React |
|-|-----------------------|------------------------|----------------------|
| Rendering | Razor views, server rendered | Angular SPA | React SPA |
| Build | Gulp bundling | Angular CLI | Vite |
| Component library | jQuery plugins + Metronic | ngx-bootstrap, ng-zorro-antd | Ant Design, React Bootstrap |
| Static hosting | Not applicable | Supported (Azure Storage, IIS, Docker) | Supported (Azure Storage, IIS, Docker) |

See the **development guide** documents for the details of all features in the different project types:

- [ASP.NET Core & jQuery](https://docs.aspnetzero.com/en/aspnet-core-mvc/latest/Features-Mvc-Core)
- [ASP.NET Core & Angular](https://docs.aspnetzero.com/en/aspnet-core-angular/latest/Features-Angular)
- [ASP.NET Core & React](https://docs.aspnetzero.com/en/aspnet-core-react/latest/Features-React)
