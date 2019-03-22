# Public Web Site

ASP.NET Zero contains a separated application that can be a starting point for your public web site or a landing page for your application. Before running the project, we need to run a NPM task to bundle and minify the CSS and JavaScript files. In order to do that, we can open a command prompt, navigate to root directory of ***.Web.Public** project and run **npm run create-bundles** command. This command should be run when a new NPM package is being added to the solution. Or you can just build your solution and all bundles will be updated automatically.

<img src="D:/Github/documents/docs/en/images/frontend-homepage.jpg" alt="Frontend home page" class="img-thumbnail" width="500" height="496" />

There are two pages here: **Home Page** and **About**. Contents of these pages are just placeholders and for demo purposes. You can completely remove content and build your page upon your needs. Also, you should change the **logo** with your Company's logo.

See [metronic front-end theme](http://keenthemes.com/free-bootstrap-templates/multi-purpose-corporate-frontend-themefreebie-corporate-frontend-theme/) for all possibilities and components to build a richer web site.

Menus are defined in **FrontEndNavigationProvider** class. When you add a new menu item here, it will be automatically shown in the menu. There is a **Login** link at the top right corner. This link takes us to the **Login page** to enter to the **backend** application.

## Layout

Layout of front-end pages are located under **Views/Layout** folder:

<img src="D:/Github/documents/docs/en/images/frontend-layout-views-core.png" alt="Frontend layout views" class="img-thumbnail" width="193" height="253" />

**\_Layout** is the main layout file that includes scripts and styles. Language flags and the menu is rendered in **Header component** which is located under Shared/Components. \_PreFooter is not used but you can add
it to the \_Layout if you want.

## New Tenant Registration

When you click "**New Tenant**" link (at the top right area) you are redirected to the edition/plan selection page:

<img src="D:/Github/documents/docs/en/images/new-tenant-select-edition-1.png" alt="New Tenant Register Edition Selection" class="img-thumbnail" />

Actually, this UI is located in the main application. When you pick an edition, you are redirected to the payment or register form depending on the button you clicked. Register form is shown as below:

<img src="D:/Github/documents/docs/en/images/tenant-signup-v3.png" alt="Tenant register form" class="img-thumbnail" />

## Single Sign On

Public web site has a login integration to the main application. When you click to login button (at the top right area) you are redirected to the main application. If you are already logged then you automatically
login in the public web site too. If not, you can enter your username and password to login. Then you are redirected back to the public web site and your username is shown at the top right:

<img src="D:/Github/documents/docs/en/images/public-web-site-login-username.png" alt="public web site login username" class="img-thumbnail" width="256" height="140" />

To make this working, public web site and main application must know their URLs. There are two configuration for that:

1. In the **appsettings.json** of the **Web.Public** project, set **AdminWebSiteRootAddress** to root URL of the main application.
2. In the **appsettings.json** of the **Web.Mvc** project, set **RedirectAllowedExternalWebSites** to root URL of the public web site.

Both configuration is properly set for the development environment. You should change them when you publish your project.