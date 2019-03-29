# Overview

## Introduction

Before reading this document, it's suggested to run the application and explore the user interface as described in the [Getting Started document](Getting-Started-Angular.md). This will help you to have a better understanding of the concepts defined here.

### About the Server Side

This document is only for the **Angular** application. For **server side**, see [ASP.NET Core documentation](Index-Core-Mvc)
too.

### IDE & Operating System

It's suggested to use an IDE to develop your project. We suggest [Visual Studio 2017+](https://www.visualstudio.com), but you can use Visual Studio Code or any other IDE/Editor you like. You can also use any OS (MacOS/Linux/Windows).

## Architecture

The diagram below shows the overall architecture of the solutions:

<img src="images/angular2-core-overall-architecture.png" alt="Angular ASP.NET Core Architecture Overall" class="thumbnail" width="683" height="163" />

Angular project is designed so that it can be **deployed separately** from the backend ASP.NET Core solution, to a different port in the same server or to a different server. When it's deployed, it's actually a
plain HTML+JS+CSS application that can be served on any operating system and any web server.

Note that ASP.NET Core solution does not have any HTML, JS or CSS code. It simply provides end points for token based authentication and to use the application services (REST APIs).

## Angular Solution

Entry point of the Angular solution is src/**main.ts**. It simply bootstraps the root [Angular Module](https://angular.io/docs/ts/latest/guide/ngmodule.html):
**RootModule**. Fundamental modules of the solution are shown below:

<img src="images/ng2-modules.png" alt="Angular 2 modules" class="img-thumbnail" width="501" height="221" />

-   **RootModule** is responsible to bootstrap the application.
-   **AccountModule** provides login, two factor authentication, register, password forget/reset, email activation, etc... It's [lazy loaded](https://angular.io/docs/ts/latest/guide/router.html).
-   **AppModule** is just to group application modules and provide a base layout. It contains two sub modules:
    -   **AdminModule** contains pages like user management, role management, tenant management, language management, settings and so on. It's lazy loaded.
    -   **MainModule** is the main module to develop your own application. It only contains a demo dashboard page which you can modify or delete. It's suggested to divide your application into smaller modules like we did in the startup project, instead of adding all functionality into the main module. This is also lazy loaded.

Fundamental modules have their own **routes**. For example; AccountModule views start with "**/account**" (like "/account/login"), AdminModule views starts with  **/app/admin**" (like "/app/admin/users").

Angular's router lazy loads modules based on their url. For instance, when you request a url starts with "app/admin", the AdminModule and all it's components are loaded. They are not loaded if you don't request those pages. That brings better startup time (and also better development time since they are independently splitted to chunks).

In addition to those fundamental modules, there are some shared modules:

-   app/shared/common/**app-common.module**: a common module used by main and admin modules as shared functionality.
-   shared/common/**common.module**: A common module used by account and app modules (and their sub modules).
-   shared/utils/**utils.module**: Another common module used by all modules (and their sub modules). We tried to collect general purpose code here those can be used even in different applications.
-   shared/service-proxies/**service-proxy.module**: Auto generated `nswag` code. It's used to communicate to backend ASP.NET Core API. We will see "how to generate automatic proxies" later.

### Configuration

Angular solution contains src/assets/**appconfig.json** file which contains some fundamental settings for the client side:

- **remoteServiceBaseUrl**: Used to configure base address of the server side APIs. Default value: http://localhost:22742
- **appBaseUrl**: Used to configure base address of the client application. Default value: http://localhost:4200
- **localeMappings**: Used to configure localizations of third-party libraries those are incompatible with existing localizations.

**appBaseUrl** is configured since we use it to define format of our URL. If we want to use tenancy name as subdomain for a multi-tenant application then we can define **appBaseUrl** as

````
http://{TENANCY_NAME}.mydomain.com
````

*{TENANCY\_NAME}* is the place holder here for tenant names. Tenancy name can also be configured for **remoteServiceBaseUrl** as similar. To make tenancy name subdomains properly work, we should also make two configurations beside the application:

1.  We should configure DNS to redirect all subdomains to a static IP address. To declare 'all subdomains', we can use wildcard like **\*.mydomain.com**.
2.  We should configure IIS to bind this static IP to our application.

There may be other ways of doing it but this is the simplest way.

> In the development time, you don't need to use subdomains for tenants for a simpler development experience. When you do like that, a 'tenant switch' dialog is used to manually switch between tenants.

### AppComponentBase

If you inherit your components from **AppComponentBase** class, you can get many commonly used services as pre-injected (like localization, permission checker, feature checker, UI notify/message, settings and so on...). For example; you can just use **l(...)** function (lowercase of 'L') In views and **this.l(...)** function in component classes for localization. See pre-built components for example usages.

## Next

- Web Application
  - [Features](Features-Angular.md)
  - [Development Tutorial](Developing-Step-By-Step-Angular-Introduction.md)
  - [Deployment](Deployment-Angular.md)
- Mobile (Xamarin) Application
  - [Development Guide](Development-Guide-Xamarin.md)
  - [Development Tutorial](Developing-Step-By-Step-Xamarin)