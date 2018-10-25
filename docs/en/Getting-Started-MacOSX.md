### Running ASP.NET Zero on Mac

 -  Install Visual Studio for Mac: https://www.microsoft.com/net/download/macos
 -  Install Visual Studio Code: https://code.visualstudio.com/
 -  Install .net core SDK: https://www.microsoft.com/net/download/macos

From ASP.Net Zero download the ASP.NET CORE & Angular latest project version (5.4.1 at this writing) with .NET Core 2.0 as chosen framework and do not check one solution.

Install latest:
 -  yarn, in my case version 1.6.0  (https://yarnpkg.com/lang/en/docs/install/#mac-stable0, I used HomeBrew)
 -  and I use nvm with node version 8.11.1  (https://github.com/creationix/nvm)
 -  and angular cli (https://cli.angular.io/)
  
Then, In the terminal, go to base_folder/angular and

	> yarn

Next open app in Visual Studio for Mac.  For starters, I opened the Web solution only, under base_folder/aspnet-core

Set Web.Host project as Startup Project (right click on Web.Host project in Solution Explorer and you will see the option)

Build all.

I used a SQL database on Azure, set up in Azure Portal, and there got connection string like this, into base_folder/appsettings.json:  

	"ConnectionStrings": {
	      "Default": "Server=tcp:research1server.database.windows.net,1433;Initial Catalog={my db name};Persist Security Info=False;User ID={my_id};Password={my password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"}, 
	
Note you have to get your IPv4 address (e.g. https://www.whatismyip.com/) and in Azure Portal  click on your database, then the "Set server firewall" button, then create a rule for your IP address (or range of addresses) and Save.  Otherwise when you start-up you will see a Connection Refused error in the browser console.

Next we open app in Visual Studio for Mac.  For starters, I opened the Web Solution only, under base_folder/aspnet-core

Set Web.Host project as Startup Project (right click on Web.Host project in Solution Explorer and you will see the option)

Now we want to get EF for dotnet. 

Go here: https://docs.microsoft.com/en-us/ef/core/miscellaneous/cli/dotnet and see Installing the Tools section. 

Edit the Web.Host project file (right click project name and there is an Tools->Edit File option) and add following:

	<ItemGroup> <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0" /> </ItemGroup>
	
Then back in Terminal:

	> dotnet add package Microsoft.EntityFrameworkCore.Design 
	> dotnet restore

Check your dotnet ef install:

	> dotnet ef

And you should see a nice ASCII unicorn.


Run the project in Visual Studio for Mac, it should take you to http://localhost:5000/swagger/ and that worked for me.

Then go to base_folder/angular and:

	> npm start
	
Then navigate in browser to http://localhost:4200/

And first it did not work for me because I had to set up the firewall rule for Azure, as above.

Then it did not work for me because the client side, according to the browser console, was trying to go to port 22742.  But my server side was going to 5000 even though I did not set that anywhere, and appsettings.json seemed to only refer to 22742.

So using Visual Studio Code, I searched the angular project and replaced all instances of 22742 with 5000.

Then… it finally worked!  (-:

There is no Package Manager Console in Visual Studio for Mac, so in Terminal you can Add-Migration:

	> dotnet ef migrations add InitialCreate

Or Update-Database:

	> dotnet ef database update
	
When you need to update service-proxies.ts then from angular directory after/while Web.Host project is up and running:

	/node_modules/.bin/nswag run


For RAD tool on Mac, there is no Visual Studio extension to create the JSON input file, you create it manually. Which can be faster actually than field by field in GUI. Then run:

	> dotnet AspNetZeroRadTool.dll YourEntity.Json

(from https://aspnetzero.com/Documents/Development-Guide-Rad-Tool-Mac-Linux)


If you want to use VS Code on Mac without Visual Studio for Mac, this might be useful to set Startup Project, although it already seems out of date...
  https://stackoverflow.com/questions/46705521/how-do-i-designate-a-startup-project-in-vs-code
