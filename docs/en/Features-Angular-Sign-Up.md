# Sign Up

When we click the "**Create Account**" link (which is only available for tenants for multi-tenant applications) in the login page, a registration form is shown:

![](images/registration-form-small-1.png)

**Recaptcha** (security question) is optional. It uses Google's Recaptcha service. Recaptcha service works per domain. So, to make it properly work, you should create your own private and public keys for your domain on https://www.google.com/recaptcha and replace keys in **appsettings.json** file on the [server side](Infrastructure-Core-Mvc-Configuration) and in the **appconfig.json** in the client side.

## Next

- [Email Activation](Features-Angular-Email-Activation)