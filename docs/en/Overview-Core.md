# Overview

## Introduction


Before reading this document, it's suggested to run the application and explore the user interface as described in the [Getting Started document](Getting-Started-Core.md). This will help you to have a better understanding of the concepts defined here.


## Solution Structure (Layers)

After you create and [download](https://aspnetzero.com/Download) your project, you have a solution structure as shown below for **\*.Web.sln**:

<img src="images/solution-overall-core-5.png" alt="ASP.NET Core solution" class="img-thumbnail" />

There are two more solutions:

* **\*.Mobile.sln** contains the Xamarin projects
* **\*.All.sln** contains both mobile and web development projects.

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

ASP.NET Zero solution contains four applications:

- **Back End Application** (`Web.Mvc`): This is the main application which needs authentication to access. You will mostly work on this application to add your business logic. Backend application is built in a dedicated area, named "**App**" by default, but can be configured while you are [creating the solution](Getting-Started-Core). So, all controllers, views and models are located under the **Areas/App** folder. Related script and style files are located under **wwwroot/view-resources/Areas/App** folder, as shown below:

  <img src="images/app-folders-core.png" alt="Application folders" class="img-thumbnail" width="161" height="381" />

- **Back End API** (`Web.Host`): An application to only serve the main application as REST API and does not provide any UI.

- **Public Web Site** (`Web.Public`): This can be used to create a public web site or a landing page for your application.

- **Migration Executer** (`Migrator`): Console application that runs database migrations.

## Basic Configuration

`appsettings.json` in Web.Mvc project contains a setting **WebSiteRootAddress**, which stores the root URL of the web application:

```csharp
"WebSiteRootAddress": "https://localhost:44302/"
```

It's used to calculate some URLs in the application. So, you need to change this on production.

### Multi-Tenancy

Multi-tenancy is used to build **SaaS** (Software as a Service) applications easily. With this technique, we can deploy **single application** to serve to **multiple customers**. Each Tenant will have it's own roles, users, settings and other data. 

ASP.NET Zero's code-base is developed to be **multi-tenant**. But, it [**can be disabled**](Getting-Started-Core#configure-multi-tenancy) with a single line of configuration if you are developing a **single-tenant** application. When you disable it, all multi-tenancy stuff will be hidden. If multi-tenancy is disabled, there will be a single tenant named **Default**.

There are two types of perspective in multi-tenant applications:

- **Host**: Manages tenants and the system.
- **Tenant**: Uses the actual application features and pays for it.

For multi-tenant applications, Web Site Root URL can contain dynamic tenancy name. In that case, put {TENANCY\_NAME} as a placeholder for tenancy name like:

```csharp
"WebSiteRootAddress": "http://{TENANCY_NAME}.mydomain.com/"
```

Thus, ASP.NET Zero can automatically detect current tenant from URLs. If you configure it as above, you should also redirect all subdomains to your application. To do that;

1. You should configure DNS to redirect all subdomains to a static IP address. To declare 'all subdomains', you can use a wildcard e.g.
   **\*.mydomain.com**.
2. You should configure IIS to bind this static IP to your application.

There may be other ways of doing it but this is the simplest way.

> In the development time, you don't need to use subdomains for tenants for a simpler development experience. When you do like that, a 'tenant switch' dialog is used to manually switch between tenants.

Check out the [multi tenant documentation](https://aspnetboilerplate.com/Pages/Documents/Multi-Tenancy) if you are building multi-tenant applications.

Similar to **WebSiteRootAddress**, **ServerRootAddress** setting is also exists in `appsettings.json` in .Web.Host project.

### Angular UI

In addition, .Web.Host application contains **ClientRootAddress** which is used if this API
is used by the Angular UI. If you are not using Angular UI, you can ignore it. 

**CorsOrigins** setting is used to allow some domains for cross origin requests. This is also useful when you are hosting your Angular UI in a separated server/domain.

##  Next

- Web Application
  - [Features](Features-Mvc-Core.md)
  - [Development Tutorial](Developing-Step-By-Step-Core-Introduction.md)
  - [Deployment](Deployment-Mvc-Core.md)
- Mobile (Xamarin) Application
  - [Development Guide](Development-Guide-Xamarin.md)
  - [Development Tutorial](Developing-Step-By-Step-Xamarin)