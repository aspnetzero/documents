# Azure Key Vault Support

ASP.NET Core provides many configuration providers out of the box. One of them is Azure Key Vault Configuration Provider. **Azure Key Vault** is a cloud service that provides a secure store for secrets. You can securely store **keys**, passwords, certificates, and other secrets. For more information about Azure Key Vault, please refer to https://docs.microsoft.com/en-us/aspnet/core/security/key-vault-configuration.

ASP.NET Zero provides built-in integration for Azure Key Vault. You can easily configure Azure Key Vault for ASP.NET Zero by filling the configurations displayed below in appsettings.json of your application. For other environments like Production or Staging, use the correct setting file (appsettings.Production.json or appsettings.Staging.json). 

````json
 "Configuration": {
    "AzureKeyVault": {
      "IsEnabled": "false",
      "KeyVaultName": ""
    }
  }
````

* ```IsEnabled```: Enables or disables using Azure Key Vault configuration. 
* ```KeyVaultName```: Key Vault Name.

For more information, you can check documentation of [https://www.nuget.org/packages/Azure.Extensions.AspNetCore.Configuration.Secrets](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.Configuration.Secrets)