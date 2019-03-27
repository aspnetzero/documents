# Configuration

ASP.NET Zero is properly configured for development. But when you want to publish your application to your **test/production environment**, you may need to change some configuration in order to make it properly work. Each section in this document describes it's own configuration, but we will provide a summary here.

Every application in the solution has it's own **appsettings.json** that should be independently configured.

##### Web.Mvc Application

- "**ConnectionStrings:Default**": Database connection string.
- "**Abp.RedisCache**": Redis cache settings if you are enabled [Redis cache provider](https://aspnetboilerplate.com/Pages/Documents/Caching#redis-cache-integration).
- "**App:WebSiteRootAddress**": Root URL of this application.
- "**App:RedirectAllowedExternalWebSites**": A comma separated list of root URLs those are allowed to be redirected once user logins. For security reasons, ASP.NET Zero only redirects to local URLs except this list. If you will use the public web site, you should add it's root URL to this list.
- "**Authentication**": Authentication settings especially for external login providers.
- "**Recaptcha**": Recaptcha settings if you enabled it.
- "**IdentityServer**": IdentityServer settings. It's important to disable it if you are not using IdentityServer. If you are using, ensure that you configured proper settings.
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
- "**App:ServerRootAddress**": Root URL of this application.
- "**App:ClientRootAddress**": Root URL of the Angular application (if you are using Angular as UI).
- "**App:CorsOrigins**": Allowed origins for cross origin requests (splitted by comma).
- "**Authentication**": Authentication settings especially for external login providers.
- "**IdentityServer**": IdentityServer settings. It's important to disable it if you are not using IdentityServer. If you are using, ensure that you configured proper settings.
- "**Payment**": Payment provider settings if you are developing a paid SaaS product.