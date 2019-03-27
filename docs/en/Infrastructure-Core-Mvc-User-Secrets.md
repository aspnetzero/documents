# User Secrets

ASP.NET Core introduced [user secrets](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets) system to store **sensitive data** in development. ASP.NET Zero uses
this system (it's configured properly for your solution). You may want to use a different connection string (or social media API keys) in development and do not want to add these secret data in your `appsettings.json` in the project (and do not want to commit these sensitive information to your source control system). Then use secret manager tool to store this sensitive information in your local computer
and allow your application to read them from your local computer if available.

For example, you can use the following command, in Windows **command prompt** in the location of **Core** project, to change connection string for your local development environment:

```
dotnet user-secrets set ConnectionStrings:Default "Server=1.2.3.4;Database=MyProjectDevDb;User=sa;Password=12345678"
```

This user secret value overrides the value in the appsettings.json. See ASP.NET's [own documentation](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets) for details about user secrets.