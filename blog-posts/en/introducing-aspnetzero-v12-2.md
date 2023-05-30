**Title:** Introducing ASP.NET Zero v.12.2
**Description:** Introducing ASP.NET Zero's latest release! Xamarin to MAUI migration, bug fixes, and minor features. Power Tools: Revolutionize UI testing with streamlined automation. Effortlessly generate tests in a few clicks. Analyze UI, automate test cases, save time. Embrace efficiency, upgrade workflow, unlock ASP.NET Zero Power Tools!

## What's new in ASP.NET Zero

I wanted to share with you some of the **innovations** that I believe may be of **most interest** to you, alongside **numerous bug fixes**.

### 🎭 Replace jest-playwright with playwright 

**Playwright** is a powerful and versatile **testing framework** that provides **cross-browser** and cross-platform support for **automated** testing. It offers a **unified API** to interact with web browsers, enabling **seamless testing** across different browser environments. With Playwright, you can write **robust** and **reliable tests** that cover a wide range of **scenarios**.

By migrating from **jest-playwright** to **Playwright**, we are taking advantage of Playwright's extensive **features** and improved **performance**. Playwright offers enhanced support for modern web technologies, increased **stability**, and better **concurrency**. It also provides a broader range of **browser options**, including Chrome, Firefox, and WebKit, allowing you to test your application across **multiple browsers** easily.

Take a look for playwright documentation -> https://playwright.dev/docs/intro

For more information about our UI test doc -> https://docs.aspnetzero.com/en/aspnet-core-mvc/latest/Playwright-UI-Testing

### Power Tools

**ASP.NET Zero Power Tools** further enhance the **development process** by enabling **faster** and **easier** project creation. You can utilize the developer interface, **command-line** interface, or **Visual Studio extension** to generate your project. In addition to **core features** like **user management** and **role-based authorization**, Power Tools support crud page generation, **multi-tenancy**, **audit logging**, **exception handling**, **background jobs**, **notifications**, **settings**, **localization**, **and more**.

https://docs.aspnetzero.com/en/common/latest/Rapid-Application-Development

I'm **thrilled** to share some **exciting news** related to **ASP.NET Zero Power Tools** with all of you. 🥳

#### 🧪 Unit Test Generation

We are thrilled to announce a **new addition** to our Power Tools! We have introduced a powerful feature that allows you to **generate unit tests** for your **application services**.

Unit testing plays a **crucial role** in ensuring the **quality** and **reliability** of your software. It helps identify **potential issues** early in the development process and provides a safety net for **future code changes**. However, writing unit tests manually can be **time-consuming** and tedious, especially as your project grows.

With the latest update to our Power Tools, we have **simplified** and **automated** the **unit test generation** process. Now, you can **quickly** generate **unit tests** for your application services with just **a few clicks**. This feature intelligently **analyzes** your service **methods**, their input **parameters**, and expected **outputs**, and generates **test cases** accordingly.

#### 🎨 UI Test Generation

**Unit tests** are great for **testing** the **logic** of your application, but they **don't cover** the **user interface**. **UI tests** are a **great way** to ensure that your **application** is **working as expected**. However, **writing UI tests** can be **time-consuming** and **tedious**.

In our **latest update**, we have introduced a highly **streamlined** and **automated** **UI test generation** process. With just a few clicks, you can now **quickly** generate **comprehensive** UI tests for your **application**. This new feature intelligently analyzes the UI elements of your application and **automatically** generates **corresponding** test **cases**. **Say goodbye** to the **tedious** **manual** test **creation** process and welcome **the speed** and **efficiency** of **automated UI testing**. Embrace this **powerful enhancement** and **revolutionize** your **testing workflow** with ASP.NET Zero Power Tools!

You can run the generated UI tests **easily** with following command.

```bash
npm test tests/mvc
```

UI Test Report
![Ui Test](images/ui-test-report.png)

UI Test Report Detail
![Ui Test](images/ui-test-report-detail.png)

### 🌙 Better contract for readibility in Dark mode

We have addressed an issue that many users have raised regarding **low contrast** and **readability** in **dark mode**, particularly when it comes to text.

We **remain committed** to continuously **refining** and **optimizing** our **design** choices to provide you with a **seamless** and visually pleasing **experience** across **all modes**. Your **feedback** and **suggestions** are **invaluable** in **helping us** enhance the usability of Zero, and we **appreciate** your **contribution** to making our **product** even **better**.

### 🆔 Removed IdentityServer4 because it is deprecated

We have an important update regarding the **authentication** and **authorization** **capabilities** of our ASP.NET Zero. After careful consideration, we have made the **decision** to **remove** **IdentityServer4** from our framework due to its **deprecation** and changes in **licensing policy**.

IdentityServer4 has been a **widely used** and **trusted** open-source framework for implementing **authentication** and **authorization** in ASP.NET applications. However, **recent changes** in the **licensing policy** have made it a **non-viable** option for us to continue supporting within ASP.NET Zero.

Moving forward, you will have the **flexibility** to **choose** the **authentication server** that best fits **your requirements**. **We are** actively **working** on **providing examples** and resources for integrating **OpenIdDict**, **Auth0**, and other **popular authentication providers**. These resources will **assist** you in **implementing** a secure and **reliable** **authentication solution** for your ASP.NET Zero applications.

**Stay tuned** for **upcoming blog posts** and **documentation** that will **guide** you through the process of **integrating** alternative **authentication** providers into your ASP.NET Zero applications. We are **committed** to **assisting** you throughout this **transition** and providing you with the **necessary resources** to make the most of our product.

https://documentation.openiddict.com/guides/index.html
https://auth0.com/docs

### 📱Removed Xamarin template

We have decided to **remove** the **Xamarin** template from our product. This **decision** was made after **careful consideration** of the current state of the **Xamarin platform** and its future prospects. 

**Don't worry** 😟, though! We are still **committed** to **providing** you with the best **mobile development** experience with **MAUI**. We will continue to support our **existing Xamarin** customers and provide them with the necessary resources **to migrate** to other platforms.

![Xamarin](images/maui-application.png)

Check out for more -> https://docs.aspnetzero.com/en/aspnet-core-mvc/latest/Development-Guide-MAUI

### 🧷 Abp.HtmlSanitizer package

To prevent **XSS attacks**, it's important to sanitize HTML input of actions. The **AbpHtmlSanitizerActionFilter** is a useful tool for this purpose. Configuring the **AbpHtmlSanitizer** easier with the new extension methods.

```csharp
Configuration.Modules.AbpHtmlSanitizer()
    .KeepChildNodes()
    .AddSelector<IEditionAppService>();
```

### 📦 Upgrade .NET packages to 7.0.5

We have **upgraded** the .NET packages to 7.0.5. Keeping libraries up-to-date is important to increase **performance** and get rid of **security** vulnerabilities.

### 🙏Conclusion

We hope you enjoy these new **features** and **enhancements**. We are always looking for ways to improve our products, so please let us know if you have any **feedback** or **suggestions**.