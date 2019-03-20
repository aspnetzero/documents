# Host Settings

Host settings page is used to configure some system settings:

<img src="D:/Github/documents/docs/en/images/host-settings-general-6.png" alt="General Host Settings" class="img-thumbnail" />

**Timezone** is an important setting in this page. ASP.NET Zero can work
in multiple zones. Each user can see dates and times in their own time
zone. Timezone setting in this page allows you to set default time zone
for the application including all tenants and users. Tenants and users
can change time zone in their own settings. Timezone setting is
available only if you are using UTC clock. [See
documentation](https://aspnetboilerplate.com/Pages/Documents/Timing) to
switch to UTC.

**Security** tab in host settings page contains password complexity
settings. Host can define system wide password complexity settings in
this tab. Each tenant can override this setting in tenant settings page.

<img src="D:/Github/documents/docs/en/images/host-settings-security-3.png" alt="Tenant settings" class="img-thumbnail" />

**Email(SMTP)** tab allows you to configure smtp settings for your app. AspNet Zero uses MailKit to send emails. By default, smtp certificate validation is disabled in **YourProjectNameMailKitSmtpBuilder.cs** class. If you are able to validate mail server's certificate, you need to modify **ServerCertificateValidationCallback** in **YourProjectNameMailKitSmtpBuilder.cs**.

If you want each Tenant to configure their own SMTP settings, you can go to **YourProjectNameConsts.cs** which is in ***.Core.Shared** project and set **AllowTenantsToChangeEmailSettings** to true. In that way, each tenant can configure and use their own SMTP settings.