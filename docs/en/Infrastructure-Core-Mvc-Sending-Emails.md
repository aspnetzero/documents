# Sending Emails

ASP.NET Zero sends emails to users in several situations, for example email confirmation, password reset, passwordless login codes, offline chat messages and subscription notices. All of them are sent through `UserEmailer` in the `.Core` project.

## Email Templates

Every email is rendered from an HTML template. The templates live in the **Net/Emailing/EmailTemplates** folder of the `.Core` project and are compiled into the assembly as embedded resources:

| Resource file | Used for |
|---------------|----------|
| `email-activation.html` | Email address verification of a new user. |
| `password-reset.html` | Password reset requests. |
| `passwordless-code.html` | One-time passwordless login codes. |
| `password-changed.html` | Password change confirmations. |
| `email-change.html` | Verification of a new email address. |
| `chat-message.html` | Chat messages received while the user is offline. |
| `subscription-notice.html` | Subscription expiration, payment and edition change events. |

A template is plain HTML with `{TOKEN}` placeholders that `UserEmailer` replaces before sending.

You can edit these files directly in your own source code, but that requires a rebuild and a redeploy. Alternatively, a host administrator can customize any of them at runtime from the **Administration > Email Templates** page; the customized body is stored in the `AppEmailTemplates` table and overrides the embedded resource. See the [Email Templates](Features-Mvc-Core-Email-Templates) document for details.

`IEmailTemplateProvider` is what resolves a template: it looks in the `AppEmailTemplateCache` cache first, then in the database, and falls back to the embedded resource. It also substitutes the base variables (`{PRODUCT_NAME}`, `{THIS_YEAR}`, `{EMAIL_LOGO_URL}`, `{EMAIL_LOGO_URL_DARK}`) that apply to every template.

## Sending

ASP.NET Zero uses [MailKit](https://github.com/jstedfast/MailKit) to send emails, through ABP's `Abp.MailKit` module.

SMTP settings (host, port, user name, password, SSL, default from address) are configured on the [Host Settings](Features-Mvc-Core-Host-Settings) page and, when `AbpZeroTemplateConsts.AllowTenantsToChangeEmailSettings` is set to `true` in the `.Core.Shared` project, also on the [Tenant Settings](Features-Mvc-Core-Tenant-Settings) page of each tenant.

## Emails in Development

**Email sending is disabled in DEBUG mode**, because a development environment is usually not configured to send emails. `IEmailSender` is replaced with `NullEmailSender` in the `PreInitialize` method of the *YourProjectName*`CoreModule` class:

```csharp
if (DebugHelper.IsDebug)
{
    //Disabling email sending in debug mode
    Configuration.ReplaceService<IEmailSender, NullEmailSender>(DependencyLifeStyle.Transient);
}
```

Remove or change this block if you want to send real emails while debugging. Email sending is always enabled in RELEASE mode.
