# Sending Emails

ASP.NET Zero sends emails to users in some cases (like forgot password and email confirmation). Email template is defined in **Emailing/EmailTemplates** folder of .Core project (**default.html**). You can change default email template by editing this file. 

**Email sending** is disabled in DEBUG mode. Because, development environment may not be configured properly to send emails. You can enable it if you want. It's enabled in RELEASE mode. Check *YourProjectName*CoreModule class's PreInitialize method to change it if you like.

**.NET Core Compatibility**

Since .NET Core does not support smpt client, ASP.NET Zero uses [MailKit](https://github.com/jstedfast/MailKit) to send emails.