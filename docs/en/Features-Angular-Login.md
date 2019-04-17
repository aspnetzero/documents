# Login

Default route for AccountModule is the login/**login.component**:

<img src="images/login-screen-3.png" alt="Login page" class="img-thumbnail" />

The **tenant selection** section above login section is shown only in a **multi-tenant** application and if "tenancy name detection" is **not possible from url** (for example, if you use subdomain as tenancy names,
no need to show a tenant selection). When we click Change link, tenant change dialog appears and we can change the tenant. There is a single tenant named **Default** in the initial database. Leave tenancy name input as blank to login as **host**.

We can use **admin** user name and **123qwe** password in first run the application.

If you select "Should change password on next login" while creating a user, the user will see the change password screen. This is not selected for initially created users.

<img src="images/account-change-password-1.png" alt="Change password" class="img-thumbnail" />

After the login, we are redirected to the **application** (app.module).

## User Lockout

You can configure user lockout settings on Settings page. Users are locked out for specified duration if they enter wrong password for a specified amount of times.

Both values can be configured under the Security tab of Settings page in the application.

## Next

* [Social Logins](Features-Angular-Social-Logins)

