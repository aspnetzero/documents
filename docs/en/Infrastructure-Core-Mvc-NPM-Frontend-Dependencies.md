# NPM & Front End Dependencies

ASP.NET Zero solution uses [NPM](https://www.npmjs.com/) package manager to obtain front end library dependencies (like Bootstrap and jQuery). So, you can easily add new packages or update existing packages on command line interface. You can see all installed NPM packages in **package.json** of the ***.Web.Mvc** project.

<img src="D:/Github/documents/docs/en/images/npm-dependencies-core.png" alt="NPM dependencies" class="img-thumbnail" />

NPM installs dependencies into node\_modules folder which will be placed in the root folder of MVC project. But, in ASP.NET Core, it is suggested to place client side libraries under wwwroot folder. Also, size of node\_modules folder will be very big (more than 250 MB) and we don't want to send all of those files to production when we publish our application. In order to overcome this, we have used gulp to move necessary files from **\*.Web.Mvc/node\_modules** to **\*.Web.Mvc/wwwroot/lib**. Mapping from node_modules to wwwroot/lib folder is defined in **package-mapping-config.js** file. So, when you add a new package to your solution, you also need to add a mapping to this file defining the files you want to move from node_modules to wwwroot/lib folder for newly added package. 

Here is a sample **package-mapping-config.js** file;

<img src="D:/Github/documents/docs/en/images/gulp-bundle-config-mappings.png-2.png" alt="Gulp mappings" class="img-thumbnail" />

In order to create css and javascript bundles https://www.nuget.org/packages/BundlerMinifier.Core/ package is used. Bundling definitions are stored in **bundleconfig.json** file. If you don't want to modify this file manually, you can use https://marketplace.visualstudio.com/items?itemName=MadsKristensen.BundlerMinifier Visual Studio extension to create bundling definitions for you.