# Deployment

## About Deployment

AspNet Zero depends on [angular-cli](https://cli.angular.io/) for development & deployment. It uses angular-cli commands to build and publish its Angular client application. AspNet Zero also minifies and bundles some assets (JS and CSS files) using [gulp](https://gulpjs.com/) before publishing the Angular client application since those assets are loaded dynamically at runtime. In order to publish the Angular client application below commands must be run respectively.

1. run ```yarn``` command in the root directory of Angular project.
2. run ```npm run publish``` command in the root directory of Angular project.

AspNet Zero also provides a merged solution for ASP.NET Core & Angular version. In that solution, Angular client side application is included in the server side Host project. Those two applications, server side API project and Angular client side application, can be published at once and can be hosted together under the same website.

To publish merged Angular solution, follow the steps below;

1. run ```dotnet publish -c Release``` command in the root directory of *.Host project. You can add parameters to dotnet publish command, see its [documentation](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-publish).
2. When the publish is completed, go to the publish directory and move all files from ```wwwroot/dist``` to ```wwwroot``` folder.

Don't forget to configure settings for the Angular application in **assets/appconfig.production.json** after publishing your app. If you are using the merged Angular project, also configure settings in **appsettings.Production.json** file.

## AOT

Angular CLI uses **[AOT](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html) (Ahead of Time)** compilation by default. You can add --no-aot parameter
to the `ng build` command to disable it. But we recommend AOT feature since it has significant performance gain.

# Next

* [Publishing to Azure](Deployment-Angular-Publish-Azure)
* [Publishing to IIS](Deployment-Angular-Publish-IIS)
* [Publishing to Docker](Deployment-Angular-Docker)
