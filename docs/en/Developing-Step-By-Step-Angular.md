# Step by Step Development



## Multi Tenancy

We have built a fully functional application until here. Now, we will
see how to convert it to a multi-tenant application easily. Logout from
the application before any change.

### Enable Multi Tenancy

We disabled multi-tenancy at the beginning of this document. Now,
re-enabling it in **PhoneBookDemoConsts** class:

```csharp
public const bool MultiTenancyEnabled = true;
```

### Make Entities Multi Tenant

In a multi-tenant application, a tenant's entities should be isolated by
other tenants. For this example project, every tenant should have own
phone book with isolated people and phone numbers.

When we implement IMustHaveTenant interface, ABP automatically [filters
data](https://aspnetboilerplate.com/Pages/Documents/Data-Filters) based
on current Tenant, while retrieving entities from database. So, we
should declare that Person entity must have a tenant using
**IMustHaveTenant** interface:

```csharp
public class Person : FullAuditedEntity, IMustHaveTenant
{
    public virtual int TenantId { get; set; }

    //...other properties
}
```

We may want to add IMustHaveTenant interface to also Phone entity. This
is needed if we directly use phone repository to get phones. In this
sample project, it's not needed.

Since entities have changed, we should create a new database migration:

    Add-Migration "Implemented_IMustHaveTenant_For_Person"

This command creates a new code-first database migration. The migration
class adds an annotation this is needed for automatic filtering. We
don't have to know what it is since it's done automatically. It also
adds a **TenantId** column to PbPersons table as shown below:

```csharp
migrationBuilder.AddColumn<int>(name: "TenantId",table: "PbPersons",nullable: false,defaultValue: 1);
```

I added **defaultValue as 1** to AddColumn options. Thus, current people
are automatically assigned to **default tenant** (default tenant's id is
always 1).

Now, we can update the database again:

    Update-Database

### Run The Multi Tenant Application

**It's finished**! We can test the application. Run the project,
**login** as the **host admin** (click Change link and clear tenancy
name) shown below:

<img src="images/login-as-host-3.png" alt="Login as host" class="img-thumbnail" style="width:100.0%" />

After login, we see the **tenant list** which only contains a
**default** tenant. We can create a new tenant:

<img src="images/create-tenant-6.png" alt="Creating tenant" class="img-thumbnail" />

I created a new tenant named **trio**. Now, tenant list has two tenants:

<img src="images/tenant-management-phonebook-2.png" alt="Tenant management page" class="img-thumbnail" style="width:100.0%" />

 I can **logout** and login as **trio** tenant **admin** (Change current
tenant to trio):

<img src="images/login-as-trio1.png" alt="Login as tenant admin" class="img-thumbnail" style="width:100.0%" />

After login, we see that phone book is empty:

<img src="images/phonebook-empty1.png" alt="Empty phonebook of new tenant" class="img-thumbnail" />

It's empty because trio tenant has a completely isolated people list.
You can add people here, logout and login as different tenants (you can
login as default tenant for example). You will see that each tenant has
an isolated phone book and can not see other's people.

## Code Generation

Using new **ASP.NET Zero Power Tools**, you can speed up your development.

See [documentation](https://aspnetzero.com/Documents/Development-Guide-Rad-Tool) to learn how to use it.

## Conclusion

In this document, we built a complete example that covers most parts of
the ASP.NET Zero system. We hope that it will help you to build your own
application.

We intentionally used different approaches for similar tasks to show you
different styles of development. ASP.NET Zero provides an architecture
but does not restrict you. You can make your own style development.

### Source Code

You should [purchase](/Prices) ASP.NET Zero in order to get **source
code**. After purchasing, you can get the sample project from private
Github repository: <https://github.com/aspnetzero/aspnet-zero-samples>
