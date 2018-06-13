This document is aimed to create and run an ASP.NET Zero based project
in just 5 minutes. It's assumed that you already [purchased](/Prices)
and created your ASP.NET Zero account.

### Login

[Login](/Account/Login) to this web site with your user name and
password. Then you will see [Download](/Download) link on the main menu.

### Create a Project

Go to the [download](/Download) page. You will see a form as shown
below:

<img src="images/download-core-jquery-2.png" alt="Create project" class="img-thumbnail" />

Select **ASP.NET Core & jQuery** as Project Type and fill other required
fields. Click to the Download button, your project will be ready in one
minute. Open the **\*.Web.sln** solution in **Visual Studio 2017+**:

If you want to work on only Xamarin project, open **\*.Mobile.sln**
solution. If you want to work on both Xamarin and Web projects, open
**\*.All.sln** solution.

<img src="images/solution-overall-core-5.png" alt="ASP.NET Core solution" class="img-thumbnail" />

Right click the **.Web.Mvc** project and select "**Set as StartUp
project**": Then **build** the solution. It make take longer time in
first build since all **nuget** packages will be restored.

### Configure The Project

**Important Notice:**  

Installing client side npm dependencies using **yarn** before opening
the solution will decrease project opening & building time dramatically.

Before opening the solution open a command prompt, navigate to root
directory of **\*.Web.Mvc** project and run "**yarn**" command to
install client side dependencies.

### Prerequirements

MVC application needs the following tools to be installed:

-   [nodejs](https://nodejs.org/en/download/) 6.9+ with npm 3.10+
-   [gulp (must be installed
    globally)](https://www.npmjs.com/package/gulp)
-   [yarn](https://yarnpkg.com/)

#### Database Connection

Open **appsettings.json** in **.Web.Mvc** project and change the
**Default** connection string if you want:

    "ConnectionStrings": {
        "Default": "Server=localhost; Database=PhoneBookDemoDb; Trusted_Connection=True;"
    }

#### Database Migrations

We have two options to create and migrate database to the latest
version.

##### ASP.NET Zero Migrator Application

ASP.NET Zero solution includes a **.Migrator** (like Acme.PhoneBookDemo.Migrator) project in the solution. You can run this tool for database migrations on development and production (see [development guide](Development-Guide-Core.md) for more information).

##### Entity Framework Migration Command

You can also use Entity Framework's built-in tools for migrations. Open
**Package Manager Console** in Visual Studio, set
**.**EntityFrameworkCore as the **Default Project** and run the
**Update-Database** command as shown below: 

<img src="images/update-database-ef-core.png" alt="dotnet ef database update" class="img-thumbnail" />

This command will create your database and fill initial data. You can
open SQL Server Management Studio to check if database is created:

<img src="images/created-database-tables-4.png" alt="ASP.NET Zero Database Tables" class="img-thumbnail" />

You can use EF console commands for development and Migrator.exe for
production. But notice that; Migrator.exe supports running migrations in
multiple databases at once, which is very useful in
development/production for multi tenant applications.

#### Multi-Tenancy

ASP.NET Zero supports multi-tenant and single-tenant applications.
Multi-tenancy is **enabled by default**. If you don't have idea about
multi-tenancy or don't want to create a multi-tenant application, you
can **disable** it by setting
**AbpZeroTemplateConsts.MultiTenancyEnabled** to false in the
.Core.Shared project.

### Run The Project

Before running the project, we need to run a npm task to bundle and minify our
css and javascript files. In order to do that, we can open a command prompt, navigate to root directory of

***.Web.Mvc** project and run "**npm run create-bundles**" command. This command must be runned when a new npm package is added to the solution. Otherwise, you can just build your solution and all bundles will be updated automatically.

Now we are ready.. just run your solution. It will open login page of
your web site.

<img src="images/login-screen-3.png" alt="Login page" class="img-thumbnail" />

If multi-tenancy is enabled, you will see the current tenant and a
change link. If so, click to Change and enter **default** as tenant
name. If you leave it empty, you login as the host admin user. Then
enter **admin** as user name and **123qwe** as password. You should
change password at first login. After login to the application, you will
see the sample dashboard screen:

<img src="images/dashboardV3.png" alt="Dashboard" class="img-thumbnail" width="1235" height="965" />

### ASP.NET Zero Power Tools

You can install our rapid application development tool from the following link:

[https://marketplace.visualstudio.com/items?itemName=Volosoft.AspNetZeroPowerTools](https://marketplace.visualstudio.com/items?itemName=Volosoft.AspNetZeroPowerTools)

### More

Your solution is up and working. See [<span class="text-primary">development guide</span>](Development-Guide-Xamarin.md) for Xamarin application, [<span class="text-primary">development guide</span>](Development-Guide-Core.md) document for more information.
