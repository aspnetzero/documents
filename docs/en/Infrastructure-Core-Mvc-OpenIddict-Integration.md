# Identity Server 4 Integration

[OpenIddict](https://documentation.openiddict.com/) aims at providing a versatile solution to implement OpenID Connect client, server and token validation support in any ASP.NET Core 2.1 (and higher) application.

## Configuration

You can enable/disable or configure it from **appsettings.json** file

```json
"OpenIddict": {
    "IsEnabled": "true",
    "Applications": [{
        "ClientId": "client",
        "ClientSecret": "def2edf7-5d42-4edc-a84a-30136c340e13",
        "DisplayName": "AbpZeroTemplate_App",
        "ConsentType": "Explicit",
        "RedirectUris": ["https://oauthdebugger.com/debug"],
        "PostLogoutRedirectUris": [],
        "Scopes": [
            "default-api",
            "profile"
        ],
        "Permissions": [
            "ept:token",
            "ept:authorization",
            "gt:password",
            "gt:client_credentials",
            "gt:authorization_code",
            "rst:code",
            "rst:code id_token"
        ]
    }]
}
```

## Testing with Client

ASP.NET Zero solution has a sample console application (ConsoleApiClient) that can connects to the application, authenticates through IdentityServer4 and calls an API.



## Testing with MVC Client

You can use [aspnet-zero-samples](https://github.com/aspnetzero/aspnet-zero-samples)  -> `IdentityServerClient` project to test identity server with mvc client. 

Add a new client to `*.Web.Mvc` appsettings.json

```json
...
    {
      "ClientId": "mvcdemo",
      "ClientName": "MVC Client Demo 2",
      "AllowedGrantTypes": [ "implicit", "client_credentials" ],
      "RequireConsent": "true",
      "ClientSecrets": [
        {
          "Value": "mysecret"
        }
      ],
      "RedirectUris": [ "http://localhost:62964/signin-oidc" ],
      "PostLogoutRedirectUris": [ "http://localhost:62964/signout-callback-oidc" ],
      "AllowedScopes": [ "openid", "profile", "email", "phone", "default-api" ],
      "AllowOfflineAccess": "true"
    }
...
```

Download the `IdentityServerClient` project and open it's `Startup.cs` and modify `AddOpenIdConnect` area as seen below

```csharp
...
.AddOpenIdConnect("oidc", options =>
{
    options.SignInScheme = "Cookies";

    options.Authority = "https://localhost:44302";//change with your project url
    options.RequireHttpsMetadata = false;

    options.ClientId = "mvcdemo";
    options.ClientSecret = "mysecret";

    options.SaveTokens = true;
});
...
```



That is all. Now you can test it.

Run both projects. Go to `IdentityServerClient `project's  secure .

<img src="images/identity-server-4-test-mvc-secure.png">

It will redirect you to the login page.

<img src="images/identity-server-4-test-mvc-login.png">

After you successfully, login you will see the consent page.

<img src="images/identity-server-4-test-mvc-consent.png">

After you allow consents, you will redirect to secure page and get user claims.

<img src="images/identity-server-4-test-mvc-secure-after-login.png">

## OpenId Connect Integration

Once IdentityServer4 integration is enabled Web.Mvc application becomes an OpenId Connect server. That means another web application can use standard OpenId Connect protocol to authenticate users with your
application and get permission to share their information (a.k.a. consent screen).

## More

See [IdentityServer4's own documentation](http://docs.identityserver.io/en/latest/) to understand and configure IdentityServer4.
