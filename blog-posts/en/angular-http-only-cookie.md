# How to enable HttpOnly cookies in ASP.NET Zero Angular

HTTP-only cookies are a type of cookie that can only be accessed and manipulated by the server through HTTP requests, not by JavaScript or client-side code. They enhance security by preventing cross-site scripting (XSS) attacks that could steal sensitive information stored in cookies. 

First of all, your Host project and your Angular project must be located in the same domain. For example, if your `*.Host` project is located in `https://localhost:44300` then your Angular project must be located in `https://localhost:44300/angular`. 

## Step 1: Configure your `*Web.Core` project

Then, you need to add the following code for adding **HttpOnly** cookies in your authentication controller.

```csharp
Response.Cookies.Append("Abp.AuthToken",
    accessToken,
    new CookieOptions
    {
        Expires = Clock.Now.Add(_configuration.AccessTokenExpiration),
        HttpOnly = true,
        Secure = true,
        SameSite = SameSiteMode.None
    }
);
```

### Add HttpOnly cookies to `Authenticate` method

You can find the the `TokenAuthController` in your `*.Core` project. Find the `Authenticate` method and add the above code after the `var accessToken = CreateAccessToken( ...` line. And also you need to add refresh token cookie as well. Your code should look like this:

```csharp
// Other codes ...

    var accessToken = CreateAccessToken(
        await CreateJwtClaims(
            loginResult.Identity,
            loginResult.User,
            refreshTokenKey: refreshToken.key
        )
    );

    Response.Cookies.Append("Abp.AuthRefreshToken",
        refreshToken.token,
        new CookieOptions
        {
            Expires = Clock.Now.Add(_configuration.RefreshTokenExpiration),
            HttpOnly = true,
            Secure = true,
            SameSite = SameSiteMode.None
        }
    );

    Response.Cookies.Append("Abp.AuthToken",
        accessToken,
        new CookieOptions
        {
            Expires = Clock.Now.Add(_configuration.AccessTokenExpiration),
            HttpOnly = true,
            Secure = true,
            SameSite = SameSiteMode.None
        }
    );

    return new AuthenticateResultModel
    {
        AccessToken = accessToken,
        ExpireInSeconds = (int) TimeSpan.FromSeconds(15).TotalSeconds,
        RefreshToken = refreshToken.token,
        RefreshTokenExpireInSeconds = (int) _configuration.RefreshTokenExpiration.TotalSeconds,
        EncryptedAccessToken = GetEncryptedAccessToken(accessToken),
        TwoFactorRememberClientToken = twoFactorRememberClientToken,
        UserId = loginResult.User.Id,
        ReturnUrl = returnUrl
    };

}
```

We need to add cookies to `ImpersonatedAuthenticate` and `ExternalAuthenticate` methods in the same way as we added above. 

### Delete HttpOnly cookies in `Logout` method

We need to delete cookies in `Logout` method. Find the `Logout` method and add the following code:

```csharp
[HttpGet]
[AbpMvcAuthorize]
public async Task LogOut()
{
    // Other codes ...
    
    Response.Cookies.Delete("Abp.AuthToken");
    Response.Cookies.Delete("Abp.AuthRefreshToken");
}
```

### Create `TenantIdCookieController.cs`

We need to create a new controller for setting tenant id cookie. Create a new controller named `TenantIdCookieController` in your `*.Core` project and add the following code:

```csharp
using System;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace AngularHttpOnlyCookiesDemo.Web.Controllers;

[Route("api/[controller]/[action]")]
public class TenantIdCookieController : AngularHttpOnlyCookiesDemoControllerBase
{
    [HttpPost]
    public void SetTenantIdCookie(int tenantId)
    {
        if (tenantId == 0)
        {
            Response.Cookies.Delete("Abp.TenantId");
            
            return;
        }
        
        var cookieOptions = new CookieOptions
        {
            HttpOnly = true,
            Secure = true,
            Expires = DateTimeOffset.UtcNow.AddYears(1)
        };
        
        Response.Cookies.Append("Abp.TenantId", tenantId.ToString(), cookieOptions);
    }
}
```

## Step 2: Configure your `*.Web.Host` project

We need to handle **HttpOnly** cookies in our `*.Web.Host` project.

### Create `HttpOnlyMiddleware.cs`

We need to create a new middleware for adding **HttpOnly** cookies to the request headers. Create a new class named `HttpOnlyMiddleware` in your `*.Web.Host` project and add the following code:

```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;

namespace AngularHttpOnlyCookiesDemo.Web.Startup;

public class HttpOnlyMiddleware
{
    private readonly RequestDelegate _next;
    
    public HttpOnlyMiddleware(RequestDelegate next)
    {
        _next = next;
    }
 
    public async Task Invoke(HttpContext context)
    {
        if (context.Request.Cookies.TryGetValue("Abp.AuthToken", out var token))
        {
            context.Request.Headers.Add("Authorization", "Bearer " + token);
        }
                
        if (context.Request.Path.Value != null && context.Request.Path.Value.Contains("RefreshToken"))
        {
            context.Request.QueryString = QueryString.Create("refreshToken", context.Request.Cookies["Abp.AuthRefreshToken"] ?? string.Empty);
        }

        await _next.Invoke(context);
    }
}

public static class HttpOnlyMiddlewareExtension
{
    public static IApplicationBuilder UseHttpOnlyMiddleware(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<HttpOnlyMiddleware>();
    }
}
```

### Configure `Startup.cs`

We need to add the following code to `Startup.cs` file in your `*.Web.Host` project.

```csharp
app.Use(async (context, next) =>
{
    if (context.Request.Cookies.TryGetValue("Abp.AuthToken", out var token))
    {
        context.Request.Headers.Add("Authorization", "Bearer " + token);
    }
    
    if (context.Request.Path.Value != null && context.Request.Path.Value.Contains("RefreshToken"))
    {
        context.Request.QueryString = QueryString.Create("refreshToken", context.Request.Cookies["Abp.AuthRefreshToken"] ?? string.Empty);
    }

    await next.Invoke();
});
```

## Step 3: Configure your `Angular` project

You need to remove cookies from your Angular application and a workaround for tenant switching.

### Comment set cookie methods

We need to update `abp.js` file in your `src/assets/abp-web-resources` folder. 

Find the `abp.auth.setToken` and comment the following line:

```js
abp.auth.setToken = function (authToken, expireDate) {
    // abp.utils.setCookieValue(abp.auth.tokenCookieName, authToken, expireDate, abp.appPath, abp.domain, { 'SameSite': abp.auth.tokenCookieSameSite });
};
```

Find the `abp.auth.setRefreshToken` and comment the following line:

```js
abp.auth.setRefreshToken = function (refreshToken, expireDate) {
    // abp.utils.setCookieValue(abp.auth.refreshTokenCookieName, refreshToken, expireDate, abp.appPath, abp.domain, { 'SameSite': abp.auth.tokenCookieSameSite });
};
```

### Update `SetTenantIdCookie` Method

We need to update `SetTenantIdCookie` method in `abp.js`

```js
abp.multiTenancy.setTenantIdCookie = function (tenantId) {

    fetch('https://localhost:44301/api/TenantIdCookie/SetTenantIdCookie?tenantId='+ tenantId , {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        }
    })
    .then(data => {
        console.log('Success:', data);
    })
    .catch((error) => {
        console.error('Error:', error);
    });
};
```

### Update `tryAuthWithRefreshToken` method

Since cookies are http only, we can no longer access them via javascript, so we need to update this method.

Open `src/app/account/auth/zero-refresh-token.service.ts` and remove this line on the `tryAuthWithRefreshToken` method.

```ts 
let refreshTokenObservable = new Subject<boolean>();

let token = this._tokenService.getRefreshToken();
if (!token || token.trim() === '') {
    return of(false);
}
```

## Conclusion

That's it! Now, you can use **HttpOnly** cookies in your ASP.NET Zero Angular application.

![Angular http only cookie](images/angular-http-only-cookie.png)