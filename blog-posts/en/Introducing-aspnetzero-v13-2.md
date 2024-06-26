**Title:** Introducing ASP.NET Zero v.13.2
**Description:** ASP.NET Zero v13.2: Passwordless login, Power Tools: Custom Menu Position, Upgrade datatables.net to 2.0.1, Upgrade to ABP 9.2

# Introducing ASP.NET Zero v13.2

We are excited to announce the release of ASP.NET Zero v13.2. This release includes passwordless login, power tools custom menu position, and many other improvements. We have also updated the packages and fixed some bugs. Let's take a look at the new features and improvements.

## 🔑 Passwordless Login

Passwordless login is a new feature that allows users to log in without a password. Instead of entering a password, users receive a one-time code via email or SMS. This code is used to authenticate the user and log them in. Passwordless login is more secure than traditional password-based authentication because it eliminates the need for users to create and remember passwords. It also reduces the risk of password-related security breaches, such as phishing attacks and credential stuffing.

![Passwordless Login](/Images/Blog/passwordless-login.gif)

For more information, see the [passwordless login documentation](https://docs.aspnetzero.com/en/aspnet-core-mvc/latest/Features-Passwordless-Login)

## 🌠 Power Tools: Custom Menu Position

At least once, you have wanted to change the position of the menu items generated by Power Tools. But until now we were only supporting `root` and `administration` position. Now you can use custom positions for your entities.

![Power Tools: Custom menu position](/Images/Blog/power-tools-custom-menu-position.png)

## ⚡ Power Tools: Master-Detail Upgrades

As you also know, Power Tools is a powerful tool that generates code for you. But we have been encountering more errors than expected on master detail pages for a while now. We have fixed these errors and made improvements to the master-detail pages. Now you can use Power Tools to generate master-detail pages without any problems.

* Multi-lingual entity generation is now supported for master-detail pages.
* Excel importing & exporting is now supported for master-detail pages.

## 📈 Upgrade `datatables.net` to `2.0.1`

We have upgraded the `datatables.net` library to version `2.0.1`. With this version, we will be able to use datatables with more performance and in a more compatible way with the project. For more information, see the [datatables.net release notes](https://cdn.datatables.net/2.0.0/).

## 🌅 Upgrade to ABP 9.2

ASP.NET Zero v13.2 is based on ABP 9.2. ABP 9.2 is a major release that includes performance improvement and bug fixes. For more information, see the [ABP 9.2 release notes](https://github.com/aspnetboilerplate/aspnetboilerplate/releases/tag/v9.2).

## 🐛 Update packages and bug fixes

As we do in every version, we fixed minor padding and margin errors, localization errors, and made improvements for power tools in line with your requests.

* [https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v13.2.0-rc.1](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v13.2.0-rc.1)
* [https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v13.2.0](https://github.com/aspnetzero/aspnet-zero-core/releases/tag/v13.2.0)

## ✏️ Don't Miss Out! 

We have published new blog posts about ASP.NET Zero. You can check them out below:

* [Multi-lingual Database Design Strategies](https://aspnetzero.com/blog/mastering-multi-lingual-database-design-in-asp.net-core-with-ef-core)

* [ASP.NET Core Configuration](https://aspnetzero.com/blog/asp.net-core-configuration)

## 🙏 Conclusion

With ASP.NET Zero v13.2, we've raised the bar yet again. We hope you enjoy this release and look forward to hearing your feedback. If you have any questions or comments, please don't hesitate to contact us.

