# Login

## Account Controller

**AccountController** provides **login**, **register**, **forgot password** and **email activation** pages.

## Layout

Account management pages have a separated **\_Layout** view under **Views/Account** folder:

<img src="images/account-views-core-v2.png" alt="Account Views" class="img-thumbnail" width="216" height="214" />

Related **Script** and **Style** resources located under **view-resources/Views/Account** folder:

<img src="images/account-views-core-resources.png" alt="Account view resources" class="img-thumbnail" width="205" height="347" />

As similar, all views of the application have corresponding style and script files under **wwwroot/view-resources** folder.

## Login Page

Main view for `AccountController` is the Login page:

<img src="images/login-screen-3.png" alt="Login page" class="img-thumbnail" />

The **tenant selection** section above login section is only shown in a **multi-tenant** application and if "subdomain tenancy name detection" is **not possible** (See host settings section). When we click Change link, tenant change dialog appears and we can change the tenant. There is a single tenant named **Default** in the initial database (See Entity framework section for initial seed data). Leave tenancy name input as blank to login as **host**.

We can use **admin** user name and **123qwe** password in first run the application. At first login, we should change admin password since 123qwe is not very secure:

<img src="images/account-change-password-1.png" alt="Change password" class="img-thumbnail" />

After changing password we are redirected to the **backend application**.

### User Lockout

Users are lockout when they enter wrong password for a specified count and duration. You can configure lockout settings in the settings page of the application.

## Next

* [Social Logins](Features-Mvc-Core-Social-Logins)