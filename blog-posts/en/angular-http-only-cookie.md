# How to enable HttpOnly cookies in ASP.NET Zero Angular

Httponly cookies are a type of cookie that can only be accessed and manipulated by the server through HTTP requests, not by JavaScript or client-side code. They enhance security by preventing cross-site scripting (XSS) attacks that could steal sensitive information stored in cookies. 

First of all, your backend project and your Angular project must be located in the same domain. For example, if your Angular project is located in `https://localhost:44300`. Then your backend project `*.Host` must be located in `https://localhost:44300/api`.

## Step 1: Configure your `*Web.Core` project

Then, you need to add the following code for adding **HttpOnly** cookies in your `TokenAuthController`.

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

Find the `Authenticate` method and add the above code after the `var accessToken = CreateAccessToken( ...` line. And also you need to add refresh token cookie as well. Your code should look like this:

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

### Add HttpOnly cookies to `RefreshToken` method

We need to add get refresh token from cookie and generate new access token with it. Then, we need to add access token cookie. After changing the `RefreshToken` method, your code should look like this:

```csharp
[HttpPost]
public async Task<RefreshTokenResult> RefreshToken()
{
    // get refresh token from cookie
    var refreshToken = Request.Cookies["Abp.AuthRefreshToken"];
    
    if (string.IsNullOrWhiteSpace(refreshToken))
    {
        throw new ArgumentNullException(nameof(refreshToken));
    }

    var (isRefreshTokenValid, principal) = await IsRefreshTokenValid(refreshToken);
    if (!isRefreshTokenValid)
    {
        throw new ValidationException("Refresh token is not valid!");
    }

    try
    {
        var user = await _userManager.GetUserAsync(
            UserIdentifier.Parse(principal.Claims.First(x => x.Type == AppConsts.UserIdentifier).Value)
        );

        if (user == null)
        {
            throw new UserFriendlyException("Unknown user or user identifier");
        }

        if (AllowOneConcurrentLoginPerUser())
        {
            await _userManager.UpdateSecurityStampAsync(user);
            await _securityStampHandler.SetSecurityStampCacheItem(user.TenantId, user.Id, user.SecurityStamp);
        }

        principal = await _claimsPrincipalFactory.CreateAsync(user);

        var accessToken = CreateAccessToken(
            await CreateJwtClaims(principal.Identity as ClaimsIdentity, user)
        );
        
        // Add access token cookie
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

        return await Task.FromResult(new RefreshTokenResult(
            accessToken,
            GetEncryptedAccessToken(accessToken),
            (int) _configuration.AccessTokenExpiration.TotalSeconds)
        );
    }
    catch (UserFriendlyException)
    {
        throw;
    }
    catch (Exception e)
    {
        throw new ValidationException("Refresh token is not valid!", e);
    }
}
```

### Delete HttpOnly cookies in `Logout` method

We need to delete cookies in `Logout` method. Find the `Logout` method and add the following code:

```csharp
[HttpGet]
[AbpAuthorize]
public async Task LogOut()
{
    // Other codes ...
    
    Response.Cookies.Delete("Abp.AuthToken");
    Response.Cookies.Delete("Abp.AuthRefreshToken");
}
```

### Create `ServerCookieController.cs`

We need to create a new controller for setting tenant id cookie. Create a new controller named `ServerCookieController` in your `*.Web.Core` project and add the following code:

```csharp
using System;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace AngularHttpOnlyCookieDemo.Web.Controllers;

[Route("api/[controller]/[action]")]
public class ServerCookieController : AngularHttpOnlyCookieDemoControllerBase
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

namespace AngularHttpOnlyCookieDemo.Web.Startup;

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
            context.Request.Headers.Append("Authorization", "Bearer " + token);
        }

        await _next.Invoke(context);
    }
}

public static class HttpOnlyMiddlewareExtension
{
    public static IApplicationBuilder UseHttpOnlyToken(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<HttpOnlyMiddleware>();
    }
}
```

### Configure `Startup.cs`

We need to add the following code to `Startup.cs` file in your `*.Web.Host` project.

```csharp
app.UseHttpOnlyToken();
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

Open `src/app/account/auth/zero-refresh-token.service.ts` and update `tryAuthWithRefreshToken` method as below.

```ts 
tryAuthWithRefreshToken(): Observable<boolean> {
    let refreshTokenObservable = new Subject<boolean>();

    this._tokenAuthService.refreshToken().subscribe({
        next: () => refreshTokenObservable.next(true),
        error: () => refreshTokenObservable.next(false)
    });

    return refreshTokenObservable;
}
```

## Conclusion

That's it! Now, you can use **HttpOnly** cookies in your ASP.NET Zero Angular application.

![Angular http only cookie](images/angular-http-only-cookie.png)