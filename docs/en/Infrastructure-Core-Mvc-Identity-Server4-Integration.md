# Identity Server 4 Integration

[IdentityServer4](http://identityserver.io/) is an OpenID Connect and OAuth 2.0 framework for ASP.NET Core. ASP.NET Zero is integrated to IdentityServer4. It's **enabled by default**.

## Configuration

You can enable/disable or configure it from **appsettings.json** file

```
"IdentityServer": {
  "IsEnabled": "true",
  "Clients": [
    {
      "ClientId": "client",
      "AllowedGrantTypes": [ "password" ],
      "ClientSecrets": [
        {
          "Value": "def2edf7-5d42-4edc-a84a-30136c340e13"
        }
      ],
      "AllowedScopes": [ "default-api" ]
    },
    {
      "ClientId": "demo",
      "ClientName": "MVC Client Demo",
      "AllowedGrantTypes": [ "hybrid", "client_credentials" ],
      "RequireConsent": "true",
      "ClientSecrets": [
        {
          "Value": "def2edf7-5d42-4edc-a84a-30136c340e13"
        }
      ],
      "RedirectUris": [ "http://openidclientdemo.com:8001/signin-oidc" ],
      "PostLogoutRedirectUris": [ "http://openidclientdemo.com:8001/signout-callback-oidc" ],
      "AllowedScopes": [ "openid", "profile", "email", "phone", "default-api" ],
      "AllowOfflineAccess": "true"
    }
  ]
}
```

## Testing with Client

ASP.NET Zero solution has a sample console application (ConsoleApiClient) that can connects to the application, authenticates through IdentityServer4 and calls an API.

## OpenId Connect Integration

Once IdentityServer4 integration is enabled Web.Mvc application becomes an OpenId Connect server. That means another web application can use standard OpenId Connect protocol to authenticate users with your
application and get permission to share their information (a.k.a. consent screen).

## More

See [IdentityServer4's own documentation](https://identityserver4.readthedocs.io/en/release/) to understand and configure IdentityServer4.