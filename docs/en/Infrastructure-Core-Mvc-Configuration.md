# Configuration

ASP.NET Zero is properly configured for development. But when you want to publish your application to your **test/production environment**, you may need to change some configuration in order to make it properly work. Each section in this document describes it's own configuration, but we will provide a summary here.

Every application in the solution has it's own **appsettings.json** that should be independently configured.

##### Web.Mvc Application

- "**ConnectionStrings:Default**": Database connection string.
- "**Abp.RedisCache**": Redis cache settings if you are enabled [Redis cache provider](https://aspnetboilerplate.com/Pages/Documents/Caching#redis-cache-integration).
- "**App:Database:StartupWait**": Waits for the database server to become available while the application is starting. See [Waiting for the Database on Startup](#waiting-for-the-database-on-startup).
- "**App:WebSiteRootAddress**": Root URL of this application.
- "**App:RedirectAllowedExternalWebSites**": A comma separated list of root URLs those are allowed to be redirected once user logins. For security reasons, ASP.NET Zero only redirects to local URLs except this list. If you will use the public web site, you should add it's root URL to this list.
- "**Authentication**": Authentication settings especially for external login providers.
- "**Recaptcha**": Recaptcha settings if you enabled it.
- "**OpenIddict**": [OpenIddict](Infrastructure-Core-Mvc-OpenIddict-Integration) settings. It is disabled by default. If you enable it, replace the sample applications, client secrets and certificates with your own values.
- "**Payment**": Payment provider settings if you are developing a paid SaaS product.

**Web.Public Application**

- "**ConnectionStrings:Default**": Database connection string.
- "**App:WebSiteRootAddress**": Root URL of this application (the public web site).
- "**App:AdminWebSiteRootAddress**": Root URL of the main (Web.Mvc) application.

**Migrator Application**

- "**ConnectionStrings:Default**": Database connection string.  

**Web.Host Application**

- "**ConnectionStrings:Default**": Database connection string.
- "**Abp.RedisCache**": Redis cache settings if you are enabled [Redis cache provider](https://aspnetboilerplate.com/Pages/Documents/Caching#redis-cache-integration).
- "**App:Database:StartupWait**": Waits for the database server to become available while the application is starting. See [Waiting for the Database on Startup](#waiting-for-the-database-on-startup).
- "**App:ServerRootAddress**": Root URL of this application.
- "**App:ClientRootAddress**": Root URL of the Angular application (if you are using Angular as UI).
- "**App:CorsOrigins**": Allowed origins for cross origin requests (splitted by comma).
- "**Authentication**": Authentication settings especially for external login providers.
- "**OpenIddict**": [OpenIddict](Infrastructure-Core-Mvc-OpenIddict-Integration) settings. It is disabled by default. If you enable it, replace the sample applications, client secrets and certificates with your own values.
- "**Payment**": Payment provider settings if you are developing a paid SaaS product.

## Waiting for the Database on Startup

Your application and your database server usually start at the same time, for example when the machine reboots. If the database server is still starting up, the application can start before it and find no database to connect to. This is a common situation with SQL Server, since its service is often set to *Automatic (Delayed Start)* and can take a couple of minutes to become available.

When this happens, the application would otherwise start in a degraded state: background jobs would not be registered, host data would not be seeded, and users would be sent to the setup page even though the database is perfectly fine.

To avoid this, the application waits for the database server for a short while before it starts. If the database becomes available within that time, the application starts normally. If it does not, the application still starts (it never hangs forever) and logs a warning telling you to make the database available and restart the application.

This is configured under **App:Database:StartupWait** in `appsettings.json`:

```json
"App": {
  "Database": {
    "StartupWait": {
      "Enabled": true,
      "TimeoutSeconds": 40,
      "RetryIntervalSeconds": 3,
      "ConnectTimeoutSeconds": 5
    }
  }
}
```

- "**Enabled**": Set it to `false` to start immediately without waiting for the database.
- "**TimeoutSeconds**": How long the application waits for the database server at most.
- "**RetryIntervalSeconds**": How long it waits between two attempts.
- "**ConnectTimeoutSeconds**": How long a single connection attempt is allowed to take.

The application only waits when the database **server** cannot be reached. If the server is reachable but the database has not been created yet, it does not wait at all and takes you straight to the [setup page](Features-Mvc-Core-Setup-Page).

If your database server takes longer than the configured timeout to start, increase **TimeoutSeconds**. Note that the application does not accept requests while it is waiting, so if you host on IIS, make sure the value stays below the IIS startup time limit (`startupTimeLimit`), and increase that limit as well if you need a longer wait.