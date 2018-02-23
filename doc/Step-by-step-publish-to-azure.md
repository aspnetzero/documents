


## Create The Azure Website

Create two website in Azure. 

### Creating Website for Host

Select "**Web App + SQL**" for **Host**. 

<img src="images/azure-publish-angular-create-azure-host-website.png">

And configure it according to your needs. A sample setting is shown below:

<img src="images/azure-publish-angular-create-azure-host-website-configuration.png">

### Creating Website for AngularUI

Select "**Web App**" for **AngularUI**.

<img src="images/azure-publish-angular-create-azure-angular-website.png">

And configure it according to your needs. A sample setting is shown below:

<img src="images/azure-publish-angular-create-azure-angular-website-configuration.png">

## Publish Applications (Host and AngularUI) to The Azure

The details will be explained in the next lines. Here are the quick steps to publish applications to the Azure.

Steps to publish **Host Application** to the Azure.

- Run the migrations on the Azure
- Configure the **\*.Web.Host/appsettings.production.json**
- Publish the application to Azure

Steps to publish **AngularUI** to the Azure

- Run the `yarn` command to restore packages
- Run the `ng build -prod`
- Copy the web.config file that is placed in **angular** folder root to dist folder
- Configure the **angular/dist/assets/appconfig.json**
- Send the required files to the Azure

### Run Migrations on The Azure


