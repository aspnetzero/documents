# Login

Main view for `AccountController` is the Login page:

<img src="D:/Github/documents/docs/en/images/login-screen-3.png" alt="Login page" class="img-thumbnail" />

The **tenant selection** section above login section is only shown in a **multi-tenant** application and if "subdomain tenancy name detection" is **not possible** (See host settings section). When we click Change link, tenant change dialog appears and we can change the tenant. There is a single tenant named **Default** in the initial database (See Entity framework section for initial seed data). Leave tenancy name input as blank to login as **host**.

We can use **admin** user name and **123qwe** password in first run the application. At first login, we should change admin password since 123qwe is not very secure:

<img src="D:/Github/documents/docs/en/images/account-change-password-1.png" alt="Change password" class="img-thumbnail" />

After changing password we are redirected to the **backend application**.

## User Lockout

As seen in the previous section, you can configure user lockout settings. Users are lockout when they enter wrong password for a specified count and duration.