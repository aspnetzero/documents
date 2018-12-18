# Getting Started

This document is aimed to create and run an ASP.NET Zero based project
in just 10 minutes. It's assumed that you already [purchased](/Prices)
and created your ASP.NET Zero account.

## Login

[Login](https://aspnetzero.com/Account/Login) to this web site with your user name and
password. Then you will see [Download](https://aspnetzero.com/Download) link on the main menu.

## Create a Project

Go to the [download](https://aspnetzero.com/Download) page. You will see a form as shown
below:

<img src="images/download-angular-3.png" alt="Download angular" class="img-thumbnail" />

Select **ASP.NET Core & Angular** as Project Type and fill other
required fields. Your project will be ready in one minute. When you open
the downloaded zip file, you will see two folders:

<img src="images/angular-solution-folders.png" alt="Client Server folders" class="img-thumbnail" />

-   **angular** folder contains the [Angular
    application](Development-Guide-Angular.md) which is configured to work with [angular-cli](https://cli.angular.io/).
-   **aspnet-core** folder contains the [server side](Development-Guide-Core.md) ASP.NET Core solution and configured to work with [Visual Studio](https://www.visualstudio.com/vs/community/).

### Merging Client and Server Solutions

Client and Server solutions are designed to work separately by default
but if you want to work on a single Visual Studio solution, you can
select "One Solution" checkbox while downloading your project.

## ASP.NET Core Application

When you open Server side solution **\*.Web.sln** in **Visual Studio
2017+**, you will see the solution structure as like below:

If you want to work on only Xamarin project, open **\*.Mobile.sln**
solution. If you want to work on both Xamarin and Web projects, open
**\*.All.sln** solution.

<img src="images/aspnet-core-host-solution-4.png" alt="ASP.NET Core solution structure" class="img-thumbnail" />

Right click the **.Web.Host** project and select "**Set as StartUp
project**": Then **build** the solution. It may take a longer time
during the first build since all **nuget** packages will be restored.

### Database Connection

Open **appsettings.json** in **.Web.Host** project and change the
**Default** connection string if you want:

    "ConnectionStrings": {
        "Default": "Server=localhost; Database=PhoneBookDemoDb; Trusted_Connection=True;"
    }

### Database Migrations

We have two options to create and migrate database to the latest
version.

#### ASP.NET Zero Migrator Application

ASP.NET Zero solution includes a **.Migrator** (like
Acme.PhoneBookDemo.Migrator) project in the solution. You can run this
tool for database migrations on development and production (see
[development guide](Development-Guide-Angular.md) for more
information).

#### Entity Framework Migration Command

You can also use Entity Framework's built-in tools for migrations. Open
												 
**Package Manager Console** in Visual Studio, set *.**EntityFrameworkCore** as the **Default Project** and run the
**Update-Database** command as shown below: 

<img src="images/update-database-ef-core.png" alt="dotnet ef database update" class="img-thumbnail" />

This command will create your database and fill initial data. You can
open SQL Server Management Studio to check if database is created:

<img src="images/created-database-tables-4.png" alt="ASP.NET Zero Database Tables" class="img-thumbnail" />

You can use EF console commands for development and Migrator.exe for
production. But notice that; Migrator.exe supports running migrations in
multiple databases at once, which can be useful in
development/production for multi tenant applications.

### Multi-Tenancy

ASP.NET Zero supports multi-tenant and single-tenant applications.
Multi-tenancy is **enabled by default**. If you don't have an idea about 
Multi-Tenancy, you can read [wikipedia.org/wiki/Multitenancy](https://en.wikipedia.org/wiki/Multitenancy). If you don't want to create a multi-tenant application, you can **disable** it by
																			  
setting **AbpZeroTemplateConsts.MultiTenancyEnabled** to false in the
***.Core.Shared** project.

### Run API Host

Once you've done the configuration, you can run the application. Server
side application only contains APIs. So, default page is a swagger UI
which can be used to investigate the API:

<img src="images/swagger-ui-ng2-1.png" alt="Swagger UI" class="img-thumbnail" />

## Angular Application

### Prerequirements

Angular application needs the following tools to be installed:

-   [nodejs](https://nodejs.org/en/download/) 6.9+ with npm 3.10+
-   [Typescript 2.0+](https://www.typescriptlang.org/)
-   [yarn](https://yarnpkg.com/)

### Restore Packages

Navigate to the Angular folder, open a command line and run the following
command to restore packages:

    yarn

We suggest to use [yarn](https://yarnpkg.com/) because npm has some
problems. It is slow and can not consistently resolve dependencies, yarn
solves those problems and it is compatible to npm as well.

### Run The Application

Open the command line and run the following command:

    npm start

Once the application compiled, you can browse <http://localhost:4200> in
your browser. ASP.NET Zero also has also **HMR** (Hot Module Replacement)
enabled. You can use the following command (instead of npm start) to
enable HMR on development time:

    npm run hmr

### Login

All ready.. just run your solution to enter to the login page:

<img src="images/login-screen-3.png" alt="Login page" class="img-thumbnail" />

If multi-tenancy is enabled, you will see the current tenant and a
change link. If so, click to **Change** like and enter **default** as
tenant name. If you leave it empty, you login as the host admin user.
Then enter **admin** as user name and **123qwe** as password. You should
change your password at first login.

### Application UI

After you login to the application, you will see the sample dashboard
screen:

<img src="images/dashboardV3.png" alt="Dashboard" class="img-thumbnail" width="1235" height="965" />

## ASP.NET Zero Power Tools

You can download our rapid application development tool from the following link:

[https://marketplace.visualstudio.com/items?itemName=Volosoft.AspNetZeroPowerTools](https://marketplace.visualstudio.com/items?itemName=Volosoft.AspNetZeroPowerTools)

## More

Your solution is up and working. See [<span class="text-primary">development guide</span>](Development-Guide-Xamarin.md) for Xamarin application, [<span class="text-primary">development guide</span>](Development-Guide-Angular.md) document for more information.
