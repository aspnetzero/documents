## Creating & Running the Project

We're creating and downloading the solution named "**Acme.PhoneBookDemo**" as described in [Getting
Started](Getting-Started-Core) document. After opening solution in Visual Studio, we see an NLayered solution that consists of eight projects:

<img src="images/solution-overall-core-5.png" alt="Solution Overall" class="img-thumbnail" />

Also, run database migrations, create the database and login to the application as described in [Getting Started](Getting-Started-Core) document. After all completed and logged in to the application, we see a dashboard as shown below:

<img src="images/default-dashboard3.png" alt="Dashboard" class="img-thumbnail" />

Logout from the application for now. We will make our application **single-tenant** (we will convert it to multi-tenant later). So, we open PhoneBookDemoConsts class in the Acme.PhoneBookDemo.Core project
and disable multi-tenancy as shown below:

```csharp
public class PhoneBookDemoConsts
{
    public const string LocalizationSourceName = "PhoneBookDemo";

    public const string ConnectionStringName = "Default";

    public const bool MultiTenancyEnabled = false;

    public const int PaymentCacheDurationInMinutes = 30;
}
```

## Next

- [Adding a New Menu Item](Developing-Step-By-Step-Core-DevExtreme-Adding-New-Menu-Item.md)