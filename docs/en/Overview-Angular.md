# Overview

## Introduction

In this document, a new sample project is being created with the name "**Acme.PhoneBook**". This document is a guide while developing your project. We definitely suggest to read this document before starting to the development. Since ASP.NET Zero is built on [ASP.NET Boilerplate](https://aspnetboilerplate.com/) application framework, this document highly refers it's [documentation](https://aspnetboilerplate.com/Pages/Documents).

Before reading this document, it's suggested to run the application and explore the user interface. This will help you to have a better understanding of concepts defined here.

### About Server Side

This document is only for the **Angular** application. For **server side**, see [ASP.NET Core documentation](Index-Core-Mvc)
too.

## Pre Requirements

Following tools are needed in order to use the solution:

- [nodejs 6.9+ and Npm 3.10+](https://nodejs.org)
- [Typescript 2.0+](https://www.typescriptlang.org/)
- [yarn](https://yarnpkg.com/)

In addition, see [ASP.NET Core](Index-Core-Mvc) documentation for server side requirements and other server side features.

### IDE

It's suggested to use an IDE to develop your project. We used [Visual Studio 2017](https://www.visualstudio.com), but you can use VS Code or any other IDE/Editor you like (VS Code, for instance). You can also use any OS you like (MacOS/Linux/Windows).

## Architecture

The diagram below shows the overall architecture:

<img src="images/angular2-core-overall-architecture.png" alt="Angular ASP.NET Core Architecture Overall" class="thumbnail" width="683" height="163" />

- Angular project is designed so that it can be **deployed separately** from the backend ASP.NET Core solution, to a different port in same server or to a different server. When it's deployed, it's actually a
  plain HTML+JS+CSS web application that can be served on any operating system and any web server.

- Angular solution consists of 2 fundamental modules:

  ### Account Module

  AccountModule provides **login**, **register**, **forgot password** and **email activation** pages and located under src/account folder:

  <img src="D:/Github/documents/docs/en/images/ng2-account-module-files.png" alt="Account Module Angular 2" class="thumbnail" width="252" height="217" />

  **account.component** is the root component of account.module. **account-routing.module** defines routes for the account application.

  ### App Module

  This is the actual application module which is entered by username and password. You will mostly work on this application to add your business requirements. A screenshot from the application:

  <img src="D:/Github/documents/docs/en/images/dashboardV4.png" alt="Dashboard" class="img-thumbnail" width="1235" height="965" />

  Folder structure of the source code is like that:

  <img src="D:/Github/documents/docs/en/images/ng2-app-module.png" alt="Angular app module files" class="img-thumbnail" width="223" height="450" />

  It consists of 3 sub modules as described before. **app.component** is the root component for all views.

  

Note that ASP.NET Core solution does not have any HTML, JS or CSS code. It simply provides end points for token based authentication and to use the application services.

## Angular Solution

Entry point of the Angular solution is src/**main.ts**. It simply bootstraps the root [Angular Module](https://angular.io/docs/ts/latest/guide/ngmodule.html):
**RootModule**. Fundamental modules of the solution are shown below:

<img src="images/ng2-modules.png" alt="Angular 2 modules" class="img-thumbnail" width="501" height="221" />

-   **RootModule** is responsible to bootstrap the application.
-   **AccountModule** provides login, two factor authentication, register, password forget/reset, email activation, etc... It's [lazy loaded](https://angular.io/docs/ts/latest/guide/router.html).
-   **AppModule** is just to group application modules and provide a
    base layout. It contains 2 sub modules:
    -   **AdminModule** contains pages like user management, role
        management, tenant management, language management, settings and
        so on. It's lazy loaded.
    -   **MainModule** is the main module to develop your own
        application. It only contains a demo dashboard page which you
        can modify or delete. It's suggested to divide your application
        into smaller modules like we did in the startup project, instead
        of adding all functionality into the main module. It's lazy
        loaded.

Fundamental modules have their own **routes**. For example;
AccountModule views start with "**/account**" (like "/account/login"), AdminModule views starts with  **/app/admin**" (like "/app/admin/users"). Angular's router lazy loads modules based on their url. For instance, when you request a url starts with "app/admin", the AdminModule and all it's components are loaded. They are not loaded if you don't request those pages. That brings better startup time (and also
better development time since they are independently splitted to chunks).

In addition to those fundamental modules, there are some share modules:

-   app/shared/common/**app-common.module**: a common module used by main and admin modules as shared functionality.
-   shared/common/**common.module**: A common module used by account and app modules (and their sub modules).
-   shared/utils/**utils.module**: Another common module used by all modules (and their sub modules). We tried to collect general purpose code here those can be used even in different applications.
-   shared/service-proxies/**service-proxy.module**: Auto generated `nswag` code. It's used to communicate to backend ASP.NET Core API. We will see "how to generate automatic proxies" later.

### Configuration

Angular solution contains src/assets/**appconfig.json** file which contains some configuration for the client side:

- **remoteServiceBaseUrl**: Used to configure base address of the server side APIs. Default value: http://localhost:22742
- **appBaseUrl**: Used to configure base address of the client application. Default value: http://localhost:4200
- **localeMappings**: Used to configure localizations of third-party libraries that are incompatible with existing localizations.

**appBaseUrl** is configured since we use it to define format of our URL. If we want to use tenancy name as subdomain for a multi-tenant application then we can define **appBaseUrl** as

http://**{TENANCY\_NAME}**.mydomain.com

{TENANCY\_NAME} is the place holder here for tenant names. Tenancy name can also be configured for **remoteServiceBaseUrl** as similar. To make tenancy name subdomains properly work, we should also make two configurations beside the application:

1.  We should configure DNS to redirect all subdomains to a static IP address. To declare 'all subdomains', we can use wildcard like **\*.mydomain.com**.
2.  We should configure IIS to bind this static IP to our application.

There may be other ways of doing it but this is the simplest way.

#### AppComponentBase

If you inherit your components from **AppComponentBase** class, you can get many commonly used services as pre-injected (like localization, permission checker, feature checker, UI notify/message, settings and so on...). For example; you can just use **l(...)** function In views and **this.l(...)** function in component classes for localization. See pre-built components for example usages.

## Next

- [Features](Getting-Started-Angular-Login) to understand the solution structure and start your development.
- [Step by Step Development](Developing-Step-By-Step-Angular-Introduction) tutorial leads you to develop a multi-tenant, localized, authorized, configurable and testable application step by step.