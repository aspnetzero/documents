# Login

Default route for AccountModule is the login/**login.component**:

<img src="images/login-screen-3.png" alt="Login page" class="img-thumbnail" />

The **tenant selection** section above login section is shown only in a **multi-tenant** application and if "tenancy name detection" is **not possible from url** (for example, if you use subdomain as tenancy names,
no need to show a tenant selection). When we click Change link, tenant change dialog appears and we can change the tenant. There is a single tenant named **Default** in the initial database. Leave tenancy name input as blank to login as **host**.

We can use **admin** user name and **123qwe** password in first run the application. At first login, we should change admin password since 123qwe is not very secure:

<img src="images/account-change-password-1.png" alt="Change password" class="img-thumbnail" />

After changing password we are redirected to the **application** (app.module).

## User Lockout

As seen in the previous section, you can configure user lockout settings. Users are lockout when they enter  wrong password for a specified count and duration.

## Next

* [Social Logins](Features-Angular-Social-Logins)

