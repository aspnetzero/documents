# Tenant Management

*If you're not developing a multi-tenant application, you can skip this section.*

If this is a multi-tenant application and you logged in as a host user, then tenants page is shown:

<img src="images/tenant-management-core-3.png" alt="Tenant management page" class="img-thumbnail" />

A tenant is represented by **Tenant** class. Tenant class [can be extended](Extending-Existing-Entities.md) by adding new properties. There is an only one tenant, named **Default** as initial. **Tenancy Name** (code name) is the **unique** name of a tenant. A tenant can be **active** or **passive**. If it's passive, no user of this tenant can login to the application.

When we click the "**Create New Tenant**" button, a dialog is shown:

<img src="images/tenant-create-modal-1.png" alt="Tenant Creation Modal" class="img-thumbnail" />

**Tenancy name** should be unique and not contain spaces or other special chars since it may be used as subdomain name (like tenancyname.mydomain.com. See the section below). **Name** can be anything. **Admin email** is used as email address of the admin user of new tenant. Admin user is automatically created with the tenant. We can set a random password for admin and send activation email. When user
first logins, he/she should change the password. We can uncheck this to enter a known password.

When we create a new tenant, we should select/create a database to store new tenant's data. We can select '**Use host database**' to store tenant data in host database (can be used for single database approach) or we can specify a connection string to create/use a **dedicated database** for new tenant. ASP.NET Zero supports **hybrid** approach. That means you can use host database for some tenants and create dedicated databases for some other tenants. Even you can **group** some tenants in a separated database.

Once you assign an edition to the tenant, you can select an expiration date (see edition management section to know what happens after subscription expiration).

All tenant actions are handled by **TenantAppService** class. Example (deleting a tenant):

```
[AbpAuthorize(AppPermissions.Pages_Tenants_Delete)]
public async Task DeleteTenant(EntityRequestInput input)
{
    var tenant = await TenantManager.GetByIdAsync(input.Id);
    CheckErrors(await TenantManager.DeleteAsync(tenant));
}
```

TenantAppService mostly uses **TenantManager** domain service for tenant operations.

## Tenant Edition and Features

An **edition** can be **assigned** to a tenant (while creating or editing). Tenant will inherit all features of the assigned edition. But we can also override features and values for a tenant. Click **actions/change features** for a tenant to **customize** it's features:

<img src="images/tenant-features-core-1.png" alt="Tenant features" class="img-thumbnail" />

## Tenant User Impersonation

As a host user, we may want to perform operations in behalf of a tenant. In this case, we can click the "**Login as this tenant**" button in the actions. When we click it, we see **a modal to select a user** of the tenant. We can select any user and perform operations allowed that user. See **User Impersonation** section in this document for more information.

## Using Tenancy Name As Subdomain

A multi-tenant application generally uses subdomain to identify current tenant. **tenant1**.mydomain.com, **tenant2**.mydomain.com and so on. ASP.NET Zero automatically identify and get tenant name from subdomain (See host settings section).

## Next

- [Host Dashboard](Features-Mvc-Core-Host-Dashboard)