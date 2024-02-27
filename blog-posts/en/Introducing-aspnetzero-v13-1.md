**Title:** Introducing ASP.NET Zero v.13.1
**Description:** ASP.NET Zero v13.1: Excel Importing, Common Payment System, Column selection for user excel exporting, Using System.Text.Json` to replace `Newtonsoft, Upgrade to ABP 9.1

# Introducing ASP.NET Zero v13.1

We are excited to announce the release of ASP.NET Zero v13.1. This release includes new Power Tools features, a common payment system, and many other improvements. We have also updated the packages and fixed some bugs. Let's take a look at the new features and improvements.

## ⚡Power Tools: Excel Importing

Power Tools now supports Excel importing. You can import data from Excel files to your entities. Power Tools generate the code for importing data from Excel files for you.

When generating entities, you can select the Excel importing feature. Power Tools will generate the necessary code for you. You can then customize the generated code according to your needs.

![Power tools excel import](/Images/Blog/power-tools-excel-import.png)

## 💰 Common Payment System

I assume you know that Aspnetzero provides you [tenant's subscription management & recurring payments system](https://docs.aspnetzero.com/en/aspnet-core-mvc/latest/Features-Mvc-Core-Subscription). But what if you want to get payments easily with any product?

Now, ASP.NET Zero provides a common payment system to get payments easily. Accept payments from your customers through **PayPal**, **Stripe** or with a provider that you can easily implement yourself. You can use the common payment system for any product or service.

For more information, see the [Common Payment System](https://docs.aspnetzero.com/en/aspnet-core-mvc/latest/Features-Mvc-Core-Common-Payment-System).

## ✅ Column selection for user excel exporting

You can now select the user's columns you want to export to Excel in the grid and export them to Excel. 

In the coming versions, we will provide this for the entities you generate by using the Power Tools.

![User excel export](/Images/Blog/users-select-columns-to-excel-export.gif)

## 📝 Using `System.Text.Json` to replace `Newtonsoft`

We have replaced `Newtonsoft.Json` with `System.Text.Json` in the ASP.NET Zero v13.1. `System.Text.Json` is a JSON library that is built-in to .NET Core. It is faster and has better performance than `Newtonsoft.Json`. It is also more secure and has better support for the latest features of C#.

For more information, see the official documentation. 

https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/migrate-from-newtonsoft?pivots=dotnet-8-0

## 🌅 Upgrade to ABP 9.1

ASP.NET Zero v13.1 is based on ABP 9.1. ABP 9.1 is a major release that includes performance improvement and bug fixes. For more information, see the [ABP 9.1 release notes](https://github.com/aspnetboilerplate/aspnetboilerplate/releases/tag/v9.1).

## 🐛 Update packages and bug fixes

As we do in every version, we fixed minor padding and margin errors, localization errors, and made improvements for power tools in line with your requests.

https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v13.1.0-rc.1
https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v13.1.0

## ✏️ Don't Miss Out! 

We have published new blog posts about ASP.NET Zero. You can check them out below:

* [Easily integrate Azure Active Directory with ASP.NET Zero](https://aspnetzero.com/blog/integrating-azure-active-directory-with-asp.net-zero)


* [How to secure your application with HTTP Only Cookies in ASP.NET Zero Angular UI](https://aspnetzero.com/blog/http-only-cookies-in-asp.net-zero-angular-ui)

* [For those who love Kendo UI, you can check out our blog post about how to integrate Kendo UI with ASP.NET Zero.](https://aspnetzero.com/blog/how-to-integrate-kendoui-angular-with-asp.net-zero)

## 🙏 Conclusion

With ASP.NET Zero v13.1, we've raised the bar yet again. We hope you enjoy this release and look forward to hearing your feedback. If you have any questions or comments, please don't hesitate to contact us.

