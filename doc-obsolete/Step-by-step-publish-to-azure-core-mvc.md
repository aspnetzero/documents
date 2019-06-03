
## Introduction

Before reading this document, it's suggested to read [Getting Started](https://aspnetzero.com/Documents/Getting-Started-Core) to run the application and explore the user interface. This will help you to have a better understanding of concepts defined here.

## Create The Azure Website

Create two websites in Azure. One for **Web.Mvc** and other for **Web.Public**

### Creating an Azure Website for Web.Mvc

Select the "**Web App + SQL**" for **Web.Mvc**. 

<img src="images/azure-publish-angular-create-azure-host-website.png">

And configure it according to your needs. A sample setting is shown below:

<img src="images/azure-publish-core-mvc-create-azure-admin-website-configuration.png">

### Creating an Azure Website for Public Website

Select the "**Web App**" for the **Public Website**.

<img src="images/azure-publish-angular-create-azure-angular-website.png">

And configure it according to your needs. A sample setting is shown below:

<img src="images/azure-publish-core-mvc-create-azure-public-website-configuration.png">

## Publish Web.Mvc Application to The Azure

The details will be explained in the next lines. Here are the quick steps to publish the **Web.Mvc Application** to the Azure.

- Run the `npm run create-bundles` to bundle and minify the js/css files
- Run the migrations on the Azure
- Configure the **.Web.Mvc/appsettings.production.json**
- Publish the application to Azure

### Run the `npm run create-bundles`

Run the `npm run create-bundles` to bundle and minify the js/css files.

### Run Migrations on The Azure

One of the best ways to run migrations on the Azure is running `update-database` command in the Visual Studio. 
But this command won't run. Your client IP address should have access to the Azure. 

#### Configuring the Firewall for Client Access 

**The easiest way:** Open Management Studio and write the Azure database settings, then click connect. 
If you are already logged in to the Azure, following info screen will be shown (if you aren't already logged in, a form will be displayed before the following screen to logging in):

<img src="images/azure-publish-angular-allow-ip-to-azure.png">

Now our client IP address have access to the Azure. Of cource, this operation can also be done via the [Azure Portal](https://portal.azure.com). Check [here](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-firewall-configure) to learn how to configure the firewall for client access via Azure Portal.

#### Apply Migrations

Open **appsettings.json** in **.Web.Mvc** project and change connection settings according to the Azure Database:

<img src="images/azure-publish-angular-connection-string.png">

Open Package Manager Console in Visual Studio, set **.EntityFrameworkCore** as the Default Project and run the `update-database` command as shown below:

<img src="images/azure-publish-angular-update-database.png">

### Configure the appsettings.production.json

Azure is using **appsettings.production.json** that is placed in the **Web.Mvc**, so this file should be configured like following:

<img src="images/azure-publish-core-mvc-appsttings-production.png">

### Publish

Right click the **Web.Mvc** project and select "**Publish**". Select "**Microsoft Azure App Service**" and check "**Select Existing**". Click "**Create Profile**" button.

<img src="images/azure-publish-angular-new-publish-profile.png">

Following screen will be shown:

<img src="images/azure-publish-core-mvc-select-azure-website.png">

Select "**azure-publish-demo-server**" and click "**OK**", then click "**Publish**" button. **Web.Mvc** application is live now:

<img src="images/azure-publish-core-mvc-ui-admin.png">

## Publish Public Website to The Azure

The details will be explained in the next lines. Here are the quick steps to publish the **Web.Public** to the Azure

- Update Bundles
- Configure the **.Web.Public/appsettings.production.json**
- Publish the application to Azure

### Update Bundles

Right click **Web.Public** project and select **Bundler & Minifier/Update Bundles**

<img src="images/azure-publish-core-mvc-bundle-public.png">

### Configure the appsettings.production.json

Azure is using **appsettings.production.json** that is placed in the **Web.Public**, so this file should be configured like following:

<img src="images/azure-publish-core-mvc-appsttings-production-public.png">

### Publish

Right click the **Web.Public** project and select "**Publish**". Click "**Create new profile**" under **Publish** tab. Select "**Microsoft Azure App Service**" and check "**Select Existing**" then click "**Publish**" button.

<img src="images/azure-publish-angular-new-publish-profile.png">

Following screen will be shown:

<img src="images/azure-publish-core-mvc-select-azure-website-public.png">

Select "**azure-publish-demo-public**" and click "**OK**". Public website is live now:

<img src="images/azure-publish-core-mvc-ui.png">
