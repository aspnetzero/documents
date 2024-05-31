# Integrating Microsoft Entra ID with ASP.NET Zero

Adding Microsoft Entra ID to your ASP.NET Zero project is a quick and effective way to ensure a secure authentication process. After the basic App Registration steps in the Microsoft Entra ID (AD), you will take just a few steps to implement OpenID integration in your ASP.NET Zero project.

## Microsoft Entra ID Configuration
After completing the **App Registration** process in the **Microsoft Entra ID**, you can obtain the **Client ID** information. Additionally, to acquire the **Client Secret** information, you need to navigate to the **'Certificates & Secrets'** section and create a new secret here.

## OpenId Connect Login
ASP.NET Zero provides an integrated OpenID Connect Login in addition to social logins. This configuration can be changed in the `appsettings.json` file located in `*.Core`.

Each tenant can also configure OpenID Connect settings for their own account. To enable this feature, set the `AllowSocialLoginSettingsPerTenant` value to **true** in the `appsettings.json` file. For other social login configurations, please refer to [Features Angular Core Social Logins](https://docs.aspnetzero.com/en/aspnet-core-angular/latest/Features-Angular-Social-Logins) document.

```json
 "OpenId": {
   "IsEnabled": "true",
   "ClientId": "your_application(client)_id",
   "Authority": "https://login.microsoftonline.com/common/v2.0",
   "LoginUrl": "https://login.microsoftonline.com/common/oauth2/v2.0/authorize",
   "ValidateIssuer": "false",
   "ResponseType": "code",
   "ClaimsMapping": [
     {
       "claim": "unique_name",
       "key": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier"
     }
   ]
 }
```

- `IsEnabled`: Setting the value to **true** activates **OpenID Connect**.
- `ClientId`: The **Application (client) ID** assigned for the App Registration created in the **Microsoft Entra ID**. This value enables the recognition of your application.
- `Authority`: Represents the URL of the authorization server used in the **OpenID Connect** protocol for **Microsoft Entra ID**. For detailed information, you can refer to the [Microsoft Identity Platform - OpenID Connect](https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols-oidc) documentation.
- `LoginUrl`: This URL directs users to the login page. It should match the authority URL with the appropriate **OAuth** endpoint.
- `ValidateIssuer`: If the value of this setting is **true**, it validates the **issuer** information received from the **OpenID Connect** client. However, if your application is a **multi-tenant** application and you want all users to be able to use **Microsoft Entra ID**, disable issuer validation. Note that the term **multi-tenant app** in this context refers to the one you have created on your **Microsoft Entra ID**; it is not related to **AspNet Zero's** multi-tenant feature.
- `ResponseType`: Determines the **OpenID Connect** flow type used for authentication. **code** typically indicates the usage of the **Authorization Code Flow**.
- `ClaimsMapping`: The **claim mapping** setting allows you to map any claim retrieved from **Microsoft Entra ID** to a local claim. For example, if **Microsoft Entra ID** returns a claim named `email` and you want to use it as `email_address`, you can use this setting.

After configuring the settings in **ASP.NET Zero** and **Microsoft Entra ID**, when you login as a tenant, the **OpenID Connect** button will become active.

![Login Screen with OpenIdConnect](/Images/Blog/login-screen-with-openidconnect.png)

After clicking the **OpenID Connect** button and successfully logging in with an account with the configured **Microsoft Entra ID Account**, The next screen will get other necessary information from the tenant user.

![External Login Callback Screen ](/Images/Blog/external-login-callback-screen.png)


## FAQ

- OpenIdConnectProtocolException: Message contains error: 'invalid_client', error_description: 'AADSTS7000215: Invalid client secret provided. Ensure the secret being sent in the request is the client secret value, not the client secret ID, 
    -  When encountering a similar error, ensure to check the `ClientSecret` value. It should be noted that the **value** field of the **Client Secret** created in the **Certificates & Secrets** section needs to be obtained.

- IOException: IDX20807: Unable to retrieve document from: 'Your_Authority'. HttpResponseMessage: 'StatusCode: 400, ReasonPhrase: 'Bad Request', Version: 1.1, Content: System.Net.Http.HttpConnectionResponseContent, Headers:  
    - In case of such an error, you may need to check your `Authority` value. Depending on the **Manifest** information created in **Microsoft Entra ID** and your project, you should correct this part accordingly.
