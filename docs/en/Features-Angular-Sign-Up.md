# Sign Up

When we click the "**Create Account**" link (which is only available for tenants for multi-tenant applications) in the login page, a registration form is shown:

<img src="D:/Github/documents/docs/en/images/registration-form-small-1.png" alt="Registration form" class="img-thumbnail" />

**Recaptcha** (security question) is optional. It uses Google's Recaptcha service. Recaptcha service works per domain. So, to make it properly work, you should create your own private and public keys for your domain on <https://www.google.com/recaptcha> and replace keys in **appsettings.json** file in the [server side](Development-Guide-Core.md) and in the **appconfig.json** in the client side.

## Next

- [Email Activation](Features-Angular-Email-Activation)