# Azure Key Vault Support

ASP.NET Core provides many configuration providers out of the box. One of them is Azure Key Vault Configuration Provider. **Azure Key Vault** is a cloud service that provides a secure store for secrets. You can securely store **keys**, passwords, certificates, and other secrets. For more information about Azure Key Vault, please refer to https://docs.microsoft.com/en-us/aspnet/core/security/key-vault-configuration.

ASP.NET Zero provides built-in integration for Azure Key Vault. You can easily configure Azure Key Vault for ASP.NET Zero by filling the configurations displayed below in appsettings.json of your application. For other environments like Production or Staging, use the correct setting file (appsettings.Production.json or appsettings.Staging.json).

````json
 "Configuration": {
    "AzureKeyVault": {
      "IsEnabled": "false",
      "KeyVaultName": "",
      "UseManagedIdentity": "false",
      "TenantId": "",
      "ClientId": "",
      "ClientSecret": ""
    }
  }
````

* ```IsEnabled```: Enables or disables using Azure Key Vault configuration.
* ```KeyVaultName```: Key Vault name. The application reads secrets from ```https://{KeyVaultName}.vault.azure.net/```.
* ```UseManagedIdentity```: Connects with a managed identity instead of a client secret. Default is ```false```.
* ```TenantId```: Azure tenant ID of the app registration. Only used with a client secret.
* ```ClientId```: Client ID of the app registration. With ```UseManagedIdentity```, the client ID of a user-assigned managed identity.
* ```ClientSecret```: Client secret of the app registration. Only used with a client secret.

## Connecting with a Client Secret

Leave ```UseManagedIdentity``` as ```false``` and fill ```TenantId```, ```ClientId``` and ```ClientSecret``` with the values of an app registration that can read secrets from the key vault.

## Connecting with a Managed Identity

Starting with v16.0, ASP.NET Zero can connect to Azure Key Vault with the managed identity of the Azure resource your application runs on, such as an App Service or a virtual machine. No Key Vault credential has to be stored in the configuration in this mode.

Set ```UseManagedIdentity``` to ```true```:

* To use the **system-assigned** managed identity, leave ```ClientId``` empty.
* To use a **user-assigned** managed identity, set ```ClientId``` to the client ID of that identity.

```TenantId``` and ```ClientSecret``` are not used in this mode.

````json
 "Configuration": {
    "AzureKeyVault": {
      "IsEnabled": "true",
      "KeyVaultName": "my-key-vault",
      "UseManagedIdentity": "true",
      "TenantId": "",
      "ClientId": "",
      "ClientSecret": ""
    }
  }
````

The managed identity must be allowed to read the secrets of the key vault. If the key vault uses Azure role-based access control, assign it the **Key Vault Secrets User** role. If it uses access policies, give it the **Get** and **List** secret permissions.

A managed identity is only available when the application runs on Azure. On a development machine, keep Azure Key Vault disabled or connect with a client secret.

For more information, you can check documentation of [https://www.nuget.org/packages/Azure.Extensions.AspNetCore.Configuration.Secrets](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.Configuration.Secrets) and [managed identities for Azure resources](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview).
