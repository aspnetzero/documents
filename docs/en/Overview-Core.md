# Overview

## Introduction

In this document, a new sample project is created named **Acme.PhoneBookDemo**. This document is a complete guide while developing your project. We definitely suggest to read this document before starting to the development. Since ASP.NET Zero is built on [ASP.NET Boilerplate](https://aspnetboilerplate.com/) application framework, this document highly refers it's [documentation](https://aspnetboilerplate.com/Pages/Documents).

Before reading this document, it's suggested to run the application and explore the user interface. This will help you to have a better understanding of concepts defined here.

### Prerequirements

Following tools are needed in order to use ASP.NET Zero Core solution:

- [Visual Studio 2017 v15.3.5+](https://www.visualstudio.com)
- [Typescript 2.0+](https://www.microsoft.com/en-us/download/details.aspx?id=48593)
- [Node.js 6.9+ with NPM 3.10+](https://nodejs.org/en/download/)
- [Yarn](https://yarnpkg.com/)

## Solution Structure (Layers)

After you create and [download](https://aspnetzero.com/Download) your project, you have a solution structure as shown below for **\*.Web.sln**:

There are two more solutions, **\*.Mobile.sln** contains only Xamarin application development projects and **\*.All.sln** contains both mobile and web development projects.

<img src="images/solution-overall-core-5.png" alt="ASP.NET Core solution" class="img-thumbnail" />

There are 12 projects in the solution:

- **Core.Shared** project contains `consts`, `enums` and helper classes used both in mobile & web projects.
- **Core** project contains domain layer classes (like [entities](https://aspnetboilerplate.com/Pages/Documents/Entities) and [domain services](https://aspnetboilerplate.com/Pages/Documents/Domain-Services)).
- **Application.Shared** project contains [application service interfaces](https://aspnetboilerplate.com/Pages/Documents/Application-Services#DocIApplicationServiceInterface) and [DTO](https://aspnetboilerplate.com/Pages/Documents/Data-Transfer-Objects)s.
- **Application** project contains application logic (like [application services](https://aspnetboilerplate.com/Pages/Documents/Application-Services)).
- **EntityFrameworkCore** project contains your DbContext, [repository](https://aspnetboilerplate.com/Pages/Documents/Repositories) implementations, database migrations and other Entity Framework Core specific concepts.
- **Web.Mvc** project contains the presentation/API layer (Controllers, Views, JavaScript files, styles, images and so on) for backend and frontend applications.
- **Web.Host** project does not contain any web related files like HTML, CSS or JS. Instead, it just serves the application as remote API. So, any device can consume your application as API. For more information see [Web.Host Project](Features-Mvc-Core-Web-Host-Project)
- **Web.Core** project contains common classes used by MVC and Host projects.
- **Web.Public** project is a separated web application that can be used to create a public web site or a landing page for your application. For more information see [Public Website](Public-Website).
- **Migrator** project is a console application that runs database migrations. For more information see [Migrator Console Application](Migrator-Console-Application)
- **ConsoleApiClient** project is a simple console application for performing API requests to the application  authenticated via IdentityServer4.
- **Tests** project contains unit and integration tests.

### Applications

ASP.NET Zero solution contains 4 applications:

- **Back End Application** `Web.Mvc`: This is the main application which needs authentication to access. You will mostly work on this application to add your business logic. Backend application is built in a dedicated area, named "**App**" by default, but can be determined while you are [creating the solution](Getting-Started-Core). So, all controllers, views and models are located under **Areas/App** folder. Also, related script and style files are located under **wwwroot/view-resources/Areas/App** folder, as shown below:

  <img src="images/app-folders-core.png" alt="Application folders" class="img-thumbnail" width="161" height="381" />

- **Back End API** `Web.Host`: An application to only serve the main application as API and does not provide UI.

- **Public Web Site** `Web.Public`: This can be used to create a public web site or a landing page for your application.

- **Migration Applier** `Migrator`: Console application that runs database migrations.

## Multi-Tenancy

Multi-tenancy is used to build **SaaS** (Software as a Service) applications easily. With this technique, we can deploy **single application** to serve to **multiple customers**. Each Tenant will have it's own **roles**, **users** and **settings**. 

ASP.NET Zero's all code-base is developed to be **multi-tenant**. But, it [**can be disabled**](Getting-Started-Core#multi-tenancy) with a single line of configuration if you are developing a **single-tenant** application. When you disable it, all multi-tenancy stuff will be hidden and not available. If multi-tenancy is disabled, there will be a single tenant named **Default**.

There are two types of perspective in multi-tenant applications:

- **Host**: Manages tenants and system.
- **Tenant**: Uses the application features.

Read [multi tenant documentation](https://aspnetboilerplate.com/Pages/Documents/Multi-Tenancy) if you are building multi-tenant applications.

## Website Root URL

`appsettings.json` in Web.Mvc project contains a setting **WebSiteRootAddress**, which stores the root URL of the web application:

```csharp
"WebSiteRootAddress": "http://localhost:62114/"
```



It's used to calculate some URLs in the application. So, you need to change this on deployment. For multi-tenant applications, this URL can contain dynamic tenancy name. In that case, put {TENANCY\_NAME} instead of tenancy name like:

```csharp
"WebSiteRootAddress": "http://{TENANCY_NAME}.mydomain.com/"
```

Thus, ASP.NET Zero can automatically detect current tenant from URLs. If you configure it as above, you should also redirect all subdomains to your application. To do that;

1. You should configure DNS to redirect all subdomains to a static IP address. To declare 'all subdomains', you can use a wildcard e.g.
   **\*.mydomain.com**.
2. You should configure IIS to bind this static IP to your application.

Similar to **WebSiteRootAddress**, **ServerRootAddress** setting is also exists in `appsettings.json` in .Web.Host project. In addition, .Web.Host application contains **ClientRootAddress** which is used if this API
is used by the [Angular UI](Developing-Step-By-Step-Angular-Introduction). If you are not using Angular UI, you can ignore it. Finally, **CorsOrigins** setting is used to allow some domains for cross origin requests. This is also useful when you are hosting your Angular UI in a separated server/domain.

## Account Controller

**AccountController** provides **login**, **register**, **forgot password** and **email activation** pages.

## Layout

Account management pages have a separated **\_Layout** view under **Views/Account** folder:

<img src="images/account-views-core-v2.png" alt="Account Views" class="img-thumbnail" width="216" height="214" />

Related **Script** and **Style** resources located under **view-resources/Views/Account** folder:

<img src="images/account-views-core-resources.png" alt="Account view resources" class="img-thumbnail" width="205" height="347" />

As similar, all views of the application have corresponding style and script files under **wwwroot/view-resources** folder.

##  Next

- [Features](Features-Mvc-Core) to understand the features.
- [Step by Step Development](Developing-Step-By-Step-Core-Introduction) tutorial leads you to develop a multi-tenant, localized, authorized, configurable and testable application step by step.