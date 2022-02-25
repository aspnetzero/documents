# Step By Step Publish To Azure

## Introduction

Before reading this document, it's suggested to read [Getting Started](Getting-Started-Angular) to run the application and explore the user interface. This will help you to have a better understanding of concepts defined here.

## Create The Azure Website

It is possible to publish ASP.NET Zero's Angular client app and server side Web.Host API app together or separately. In this document, we will publish both apps separately.

So, go to your Azure Portal and create two websites, one for **Web.Host** project and other one for **Angular** application.

### Creating an Azure Website for Host

On Azure portal menu, go to App Services and click "Create App Service" button. Fill the App Service creation for correctly and create the app service for admin website.

<img src="images/azure-publish-angular-create-admin-website.png">

### Creating an Azure Website for Angular

Just like we did for Host website, create a new app service for Angular website with a different name.

## Create the Database Service

On Azure portal, go to SQL databases menu and create a new empty database. If you haven't create a new Server on Azure, click "create new" link under the Server selection combobox and first create a Server.

<img src="/images/azure-publish-mvc-create-database.png">

## Publish Host Application to The Azure

Here are the quick steps to publish the **Host Application** to the Azure.

- Run the migrations on the Azure
- Configure the **.Web.Host/appsettings.production.json**
- Publish the application to Azure

### Run Migrations on The Azure

One of the best ways to run migrations on the Azure is running `update-database` command in the Visual Studio. 
In order to do that, your public IP address should have access to the Azure. 

#### Configuring the Firewall for Client Access 

**The easiest way:** Open Management Studio and write the Azure database settings, then click connect. 
If you are already logged in to the Azure, following info screen will be shown (if you aren't already logged in, a form will be displayed before the following screen to logging in):

<img src="images/azure-publish-angular-allow-ip-to-azure.png">

Now your client IP address have access to the Azure. Of course, this operation can also be done via the [Azure Portal](https://portal.azure.com). Check [here](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-firewall-configure) to learn how to configure the firewall for client access via Azure Portal.

#### Apply Migrations

Open **appsettings.json** in **.Web.Host** project and change connection settings according to the Azure Database:

<img src="images/azure-publish-angular-connection-string.png">

Open Package Manager Console in Visual Studio, set **.EntityFrameworkCore** as the Default Project and run the `update-database` command as shown below:

<img src="images/azure-publish-angular-update-database.png">

As an alternative, you can change connection string on the Migrator project in your solution and execute it instead of running Update-Database command. It is suggested to use Migrator project for migration operations.

### Configure the appsettings.production.json

Since your application will run in Production environment, Azure will use **appsettings.production.json** that is placed in the **Web.Host**, so this file should be configured like following:

<img src="images/azure-publish-angular-appsttings-production.png">

```ServerRootAddress``` value here represents the URL of your Host application's Azure URL. ClientRootAddress represents your Angular UI's URL. CorsOriign represents the allowed websites which can make request to your Host application remotely. So, in this scenario, ClientRootAddress and CorstOrigins must be equal to your Angular app's URL on Azure.

### Publish

Right click the **Web.Host** project and select "**Publish**". Select "**Microsoft Azure App Service**" and check "**Select Existing**". Click "**Create Profile**" button.

<img src="images/azure-publish-angular-new-publish-profile.png">

Following screen will be shown:

<img src="images/azure-publish-angular-select-azure-website.png">

Select "**your-host-app-service-name**" and click "**OK**", then click "**Publish**" button. **Host** application is live now:

<img src="images/azure-publish-angular-swagger-ui.png">

## Publish Angular to The Azure

Here are the quick steps to publish the **AngularUI** to the Azure

- Run the `yarn` command to restore packages
- Run the `npm run publish` command to publish Angular app.
- Copy the web.config file that is placed in **angular** folder to **dist** folder
- Configure the **angular/dist/assets/appconfig.json** with your production URLs.
- Upload all files under "angular/dist" folder to your website on Azure created for the Angular UI.

### Prepare The Publish Folder

Run the `yarn` command to restore packages and run the `npm run publish` to publish the Angular app.

### Copy the web.config

Copy the web.config file that is placed in **angular** folder to **angular/dist** folder.

### Copy the appconfig.json

Configure the **angular/dist/assets/appconfig.production.json** like following:

<img src="images/azure-publish-angular-appconfig.png">

### Upload Files to Azure

Files must be uploaded to the Azure via FTP. Transfer files from the **dist** to the **www** folder in the Azure. The folder structure should look like:

<img src="images/azure-publish-angular-filezilla.png">

Angular application is live now. Browse the http://azure-publish-demo-client.azurewebsites.net and see it works.

<img src="images/azure-publish-angular-angular-ui.png">

