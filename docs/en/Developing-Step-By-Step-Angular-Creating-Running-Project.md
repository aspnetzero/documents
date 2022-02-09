## Creating & Running The Project

Let's create and download the solution named "**Acme.PhoneBookDemo**" as described in the [Getting Started](Getting-Started-Angular) document. Please follow the getting started document, run the application, login as default tenant
admin (select `Default` as tenancy name, use `admin` as username and `123qwe` as the password) and see the dashboard below:

<img src="images/default-dashboard-6.png" alt="Dashboard" />

Log out from the application for now. We will make our application **single-tenant** (we will convert it to multi-tenant later). So, open **PhoneBookDemoConsts** class in the  **Acme.PhoneBookDemo.Core.Shared** project and disable multi-tenancy as shown below:

```c#
public class PhoneBookDemoConsts
{
    public const string LocalizationSourceName = "PhoneBookDemo";

    public const string ConnectionStringName = "Default";

    public const bool MultiTenancyEnabled = false;

    public const int PaymentCacheDurationInMinutes = 30;
    
    // Other code blocks
}
```

**Note:** If you log in before changing **MultiTenancyEnabled** to false, you might get a login error. To overcome this, you should remove cookies.

## Next

- [Adding a New Menu Item](Developing-Step-By-Step-Angular-Adding-New-Menu-Item)