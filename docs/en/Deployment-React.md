# Deployment

## About Deployment

ASP.NET Zero depends on [React-cli](https://cli.React.io/) for development & deployment. It uses React-cli commands to build and publish its React client application. ASP.NET Zero also minifies and bundles some assets (JS and CSS files) using [gulp](https://gulpjs.com/) before publishing the React client application since those assets are loaded dynamically at runtime. In order to publish the React client application below commands must be run respectively.

1. run ```yarn``` command in the root directory of React project.
2. run ```npm run publish``` command in the root directory of React project.

In the Web.Host project, run ```npm run create-bundles``` to bundle and minify JS and CSS files before final deployment.

ASP.NET Zero also provides a merged solution for ASP.NET Core & React version. In that solution, React client side application is included in the server side Host project. Those two applications, server side API project and React client side application, can be published at once and can be hosted together under the same website.

To publish merged React solution, follow the steps below;

1. run ```dotnet publish -c Release``` command in the root directory of *.Host project. You can add parameters to dotnet publish command, see its [documentation](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-publish).
2. When the publish is completed, go to the publish directory and move all files from ```wwwroot/dist``` to ```wwwroot``` folder.

Don't forget to configure settings for the React application in **assets/appconfig.production.json** after publishing your app. If you are using the merged React project, also configure settings in **appsettings.Production.json** file.

## AOT

React toolchain uses **[AOT](https://React.io/docs/ts/latest/cookbook/aot-compiler.html) (Ahead of Time)** compilation by default. You can add --no-aot parameter
to the `npm run build` command to disable it. But we recommend AOT feature since it has significant performance gain.

# Next

* [Publishing to Azure](Deployment-React-Publish-Azure)
* [Publishing to IIS](Deployment-React-Publish-IIS)
* [Publishing to Docker](Deployment-React-Docker)

