# Introduction

In this article we will use [Azure Storage Static site](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website) feature to deploy the **AngularUI** site.

## Why use Azure Storage

**AngularUI** is a static website, therefore it can be deployed in Azure storage which provides features specifically designed for static websites.
such as Custom Domains, and SSL.
Also; using Azure Storage is much [cheaper](https://azure.microsoft.com/en-us/pricing/details/storage/) than deploying an app Service on Azure.

## Steps

- Create Storage Account
- Enable Static Website feature
- **OPTIONAL** Enable Custom Domain
- Publish files to Azure Storage

### Create Storage Account

Follow this article: [Create a storage account](https://docs.microsoft.com/en-us/azure/storage/common/storage-quickstart-create-account)
and make sure that you are using at least **StorageV2 (general purpose v2)** account kind.

### Enable Static Site feature

Go to the newly created storage account and navigate to **Static website** from the side menu.

*if you do not see this option: make sure that you are using at least **StorageV2 (general purpose v2)** account kind.*

- Set **Static website**: Enabled
- Set **Index document name**: index.html
- Set **Error document path**: index.html

### OPTIONAL: Custom Domain

You can use Azure Storage Static website feature to have your Custom domain redirected to it with SSL enabled.

For that to work, you must serve the contents of the static website from Azure CDN.

Follow [this article](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-https-custom-domain-cdn) to create Azure CDN end point that will serve the contents over HTTPS on a custom domain.

Remember to update the settings in ````appsettings.json```` files below to reflect the new URL for your static site, as now they will be served from the CDN over a custom domain.

Then add the task as described in the RELEASE pipeline below, to purge the cache on every release.

#### Update appsettings.json

Update the **HOST** appsettings.json with the following:

- ClientRootAddress: replaced with the static website primary end point ex: https://xxxxxx.z33.web.core.windows.net
- CorsOrigins: add the static website primary end point

Update the **AngularUI** appsettings.json with the following:

- appBaseUrl: replaced with the static website primary end point ex: https://xxxxxx.z33.web.core.windows.net

### Publish files to Azure Storage

Once you enable the static website feature on Azure storage, it will automatically create a special container with the name **$web**.

*note you cannot change this name.*

It is assumed that you already have a *dist* folder built and ready for publishing.

#### Manual Publishing

You can manually upload your *dist* files using [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/) to the *$web* container.

#### Automated publishing using Azure Pipelines

We will create a **RELEASE** pipeline here to do the following:

- Pick up the *drop* folder from the **BUILD** pipeline or any other source
- Delete everything that exists in **$web** *(always clean the storage container before we upload a new version)*
- **OPTIONAL (with custom domain)** purge the Azure CDN cache if using a custom domain
- Publish the files to the **$web** container

#### Steps to create a Release Pipeline

- Go to [Azure DevOps](https://dev.azure.com)
- Click ````Pipelines```` from the side menu
- Click ````Releases````
- Click ````New```` > ````New release pipeline````
- Click ````Empty job````
- Click ````Add an artifact```` and select the source of the **AngularUI** dist folder
- Click on ````Stage 1```` jobs link
- Click ````+```` to add a new Task to the Agent job
- Add the following tasks *(in this order)*
  - Azure CLI
  - Azure CLI (optional with custom domain)
  - Azure File Copy
- configure the tasks settings as below

##### Azure CLI: Settings

- Name: Delete all $web container  files
- Azure Subscription: ````(select your subscription)````
- Script Location: ````Inline Script````
- Inline Script
  - ````az storage blob delete-batch --account-name [STORAGE-ACCOUNT-NAME] --source $web````

##### OPTIONAL (with custom domain): Azure CLI: Settings

- Name: Purge the Azure CDN cache end point
- Azure Subscription: ````(select your subscription)````
- Script Location: ````Inline Script````
- Inline Script
  - ````az cdn endpoint purge -n [AZURE-CDN-END-POINT-NAME] -g [AZURE-CDN-RESOURCE-GROUP-NAME] --profile-name [AZURE-CDN-PROFILE-NAME] --content-paths "/*"````

##### Azure File Copy: Settings

*note: (switch to Task Version 3 if it is not the default).*

- Name: Copy new files to $web container
- Source: ````(select the source of the drop folder)````
- Azure Subscription: ````(select your subscription)````
- Destination Type: ````Azure Blob````
- RM Storage Account: ````(storage account name)````
- Container Name: ````$web````

That's it, now you can queue a release.
