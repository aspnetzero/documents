# Deployment

## About Deployment

If you have merged Angular UI project into ASP.NET Core project then you  only need to publish your .Host project. After publish .Host project, you should copy files that are in .Host/wwwroot/dist folder to publish_folder/wwwroot. 

For example: Move files in `.Host/wwwroot/dist` to `C:\inetpub\wwwroot\my-website\wwwroot`

We are using [angular-cli](https://cli.angular.io/) for development & deployment. Angular CLI has it's own build command that can be used to build your application:

```
ng build --prod
```

This command uses **dist** folder as output. Just remember to change **assets/appconfig.json** file with your own configuration.

## AOT

Angular CLI uses **[AOT](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html) (Ahead of Time)** compilation by default. You can add --no-aot parameter
to the `ng build` command to disable it. But we recommend AOT feature since it has significant performance gain.

*Be aware that, if you don't configure email settings, some functions like new tenant registration may not work.*

# Next

* [Publishing to Azure](Deployment-Angular-Publish-Azure)
* [Publishing to IIS](Deployment-Angular-Publish-IIS)
* [Publishing to Docker](Deployment-Angular-Docker)



