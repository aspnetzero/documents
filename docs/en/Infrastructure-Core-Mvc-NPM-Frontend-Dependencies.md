# NPM & Front End Dependencies

ASP.NET Zero solution uses [NPM](https://www.npmjs.com/) package manager to obtain front end library dependencies (like Bootstrap and jQuery). So, you can easily add new packages or update existing packages on command line interface. You can see all installed NPM packages in **package.json** of the ***.Web.Mvc** project.

<img src="images/npm-dependencies-core.png" alt="NPM dependencies" class="img-thumbnail" />

In order to create css and javascript bundles ASP.NET Zero uses [Gulp](https://gulpjs.com/). Bundle definitions are stored in **bundles.json** file. For more details, see [Bundling, Minifying and Compiling](Infrastructure-Core-Mvc-Bundling-Minifying-Compiling.md) documentation.

