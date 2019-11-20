# Running ASP.NET Zero on Mac

Download an ASP.NET CORE & Angular project with .NET Core framework as described in the [getting started document](Getting-Started-Angular.md). Do not check the "One solution" option when creating your project.

## Pre-Requirements

 -  Visual Studio for Mac: [<https://visualstudio.microsoft.com/vs/mac/>](<https://visualstudio.microsoft.com/vs/mac/>)
-  .Net core SDK: [https://www.microsoft.com/net/download/macos](https://www.microsoft.com/net/download/macos)

 -  Yarn [https://yarnpkg.com/en/docs/install#mac-stable](https://yarnpkg.com/en/docs/install#mac-stable)
 -  NVM with node version 8.11.1+: [https://github.com/creationix/nvm](https://github.com/creationix/nvm)
 -  Angular-cli ([https://cli.angular.io/](https://cli.angular.io/))

## Restoring Packages

In the terminal, go to base_folder/angular and run the `yarn` command:

```bash
yarn
```

## Configuration & Build

The application is configured to work with SQL Server by default. If you want to use a different database provider, please check [https://aspnetboilerplate.com/Pages/Documents/Entity-Framework-Core#other-database-integrations](https://aspnetboilerplate.com/Pages/Documents/Entity-Framework-Core#other-database-integrations).

SQL Server database on Azure is used in this example. You can set your SQL Server database up in Azure Portal, and there got connection string like this, into **appsettings.json**: 

```json
"ConnectionStrings": {
      "Default": "Server=tcp:research1server.database.windows.net,1433;Initial Catalog={my db name};Persist Security Info=False;User ID={my_id};Password={my password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"}, 
```

> Note: You have to get your IPv4 address (e.g. [https://www.whatismyip.com/](https://www.whatismyip.com/)) and in Azure Portal  click on your database, then the "**Set server firewall**" button, then create a rule for your IP address (or range of addresses) and Save.  Otherwise when you start-up you will see a Connection Refused error in the browser console.

Open the application in Visual Studio for Mac.  If you are not going to work on Xamarin app, open the Web solution only, under base_folder/aspnet-core.

Set Web.Host project as Startup Project (right click on Web.Host project in Solution Explorer and you will see the option)

You need to get EF Core for dotnet. 

Go here: [https://docs.microsoft.com/en-us/ef/core/miscellaneous/cli/dotnet](https://docs.microsoft.com/en-us/ef/core/miscellaneous/cli/dotnet) and see Installing the Tools section. 

Open the terminal in the `EntityFrameworkCore` project folder(eg `Acme.PhoneBookDemo.EntityFrameworkCore`, The ef cli commands in this article are all executed under this folder):

	> dotnet add package Microsoft.EntityFrameworkCore.Design 
	> dotnet restore

Check your dotnet ef install:

	> dotnet ef

And you should see a nice ASCII unicorn.

After that, run the command below to create your database:

	> dotnet ef database update


Run the project in Visual Studio for Mac, it should take you to [http://localhost:5000/swagger/](http://localhost:5000/swagger/).

Go to "**base_folder/angular/src/assets/**" and change port in remoteServiceBaseUrl from "**22742**" to "**5000**" in **appconfig.json** file before running the angular application.

Then go to base_folder/angular and:

	> npm start

Then navigate in browser to http://localhost:4200/

There is no Package Manager Console in Visual Studio for Mac, so in Terminal you can Add-Migration:

	> dotnet ef migrations add InitialCreate

Or Update-Database:

	> dotnet ef database update

### ASP.NET Zero Power Tools

For ASP.NET Zero Power Tools on Mac, there is no Visual Studio extension, so you need to create the JSON input file it manually, then run:

	> dotnet AspNetZeroRadTool.dll YourEntity.Json

For more information please check 
(from [Development-Guide-Rad-Tool-Mac-Linux](Development-Guide-Rad-Tool-Mac-Linux))
