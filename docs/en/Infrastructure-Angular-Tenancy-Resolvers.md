# Tenancy Resolvers

ASP.NET Boilerplate has built in tenancy resolvers for server side, see [documentation](https://aspnetboilerplate.com/Pages/Documents/Multi-Tenancy#determining-current-tenant) for more information. But, in some scenarios, you may want to determine the current tenant on the client application. For example, ASP.NET Zero's Angular template is deployed separately from its ASP.NET Core API app. 

Because of this, ASP.NET Zero provides built-in tenancy resolvers for its Angular application. These are;

* SubdomainTenantResolver
* QueryStringTenantResolver
* CookieTenantResolver

By default, these resolvers are executed in the given order. If a tenant resolver finds the tenant, it then sets `Abp.TenantId` cookie value, so the server side app can use this to determine the current tenant as well.

## SubdomainTenantResolver

Finds the tenancy name from subdomain by using the app's configured URL. For example, if the app's URL is configured as `https://{TENANCY_NAME}.mywebsite.com` and visited url is `https://tenant1.mywebsite.com`, then tenant1 will be returned as tenancy name.

## QueryStringTenantResolver

Looks for `abp_tenancy_name` query string parameter and returns its value for tenancy name if `abp_tenancy_name` parameter is present in current query string.

## CookieTenantResolver

Looks for `abp_tenancy_name` cookie value and returns its value for tenancy name if `abp_tenancy_name` cookie value is present.

You can also implement similar classes and use them in `resolveTenancyName` method of `AppPreBootstrap.ts` to determine the current tenant.

