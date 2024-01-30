# How to enable HttpOnly cookies in ASP.NET Zero Angular

HttpOnly cookies are a type of cookie that can only be accessed and manipulated by the server through HTTP requests, not by JavaScript or client-side code. They enhance security by preventing cross-site scripting (XSS) attacks that could steal sensitive information stored in cookies. 

First of all, your backend project and your Angular project must be located in the same domain. For example, if your Angular project is located in `https://localhost:44300`. Then your backend project `*.Host` must be located in `https://localhost:44300/api`.

## Step 1: Configure your `*Web.Core` project

To implement HttpOnly cookies, we need to configure our token authentication controller. We will use the `TokenAuthController.cs` file in your `*.Web.Core` project to implement HttpOnly cookies.

### Remove Token Properties from Models

To implement HttpOnly cookies effectively, our server application must refrain from returning access token and refresh token values directly from APIs. Consequently, we must remove these properties from our models and remove any unnecessary classes associated with them.

Remove following classes from your `*.Web.Core` project:

* `RefreshTokenResult.cs`
* `ImpersonatedAuthenticateResultModel.cs`
* `SwitchedAccountAuthenticateResultModel.cs`

> Adjust the method return types to `Task` and keep the generated access tokens, which will subsequently serve as **HttpOnly** cookies.

Then, remove `accessToken` and `refreshToken` properties from `AuthenticateResultModel.cs` and `ExternalAuthenticateResultModel.cs` files.

*AuthenticateResultModel.cs*
```csharp
public class AuthenticateResultModel
{
    public bool ShouldResetPassword { get; set; }

    public string PasswordResetCode { get; set; }

    public long UserId { get; set; }

    public bool RequiresTwoFactorVerification { get; set; }

    public IList<string> TwoFactorAuthProviders { get; set; }

    public string TwoFactorRememberClientToken { get; set; }

    public string ReturnUrl { get; set; }
    
    public string c { get; set; }
}
```

*ExternalAuthenticateResultModel.cs*
```csharp
public class ExternalAuthenticateResultModel
{
    public bool WaitingForActivation { get; set; }

    public string ReturnUrl { get; set; }
}
```

### Configure `TokenAuthController.cs`

* Go to the `Authenticate` method and remove missing properties from the returned objects. **(Method contains 3 return statements)**

```csharp
// other codes...

return new AuthenticateResultModel
{
    ShouldResetPassword = true,
    ReturnUrl = returnUrl,
    c = await EncryptQueryParameters(loginResult.User.Id, loginResult.Tenant, loginResult.User.PasswordResetCode)
};

// other codes...

return new AuthenticateResultModel
{
    RequiresTwoFactorVerification = true,
    UserId = loginResult.User.Id,
    TwoFactorAuthProviders = await _userManager.GetValidTwoFactorProvidersAsync(loginResult.User),
    ReturnUrl = returnUrl
};

// other codes...

return new AuthenticateResultModel
{
    TwoFactorRememberClientToken = twoFactorRememberClientToken,
    UserId = loginResult.User.Id,
    ReturnUrl = returnUrl
};
```

* Go to the `RefreshToken` method, change the return type to `Task` and remove return value.

* Go to the `ImpersonatedAuthenticate` method, change the return type to `Task` and remove return value.

* Go to the `DelegatedImpersonatedAuthenticate` method, change the return type to `Task` and remove return value.

* Go to the `LinkedAccountAuthenticate` method, change the return type to `Task` and remove return value.

* Go to the `ExternalAuthenticate` method and remove missing properties from the returned objects. **(Method contains 3 return statements)**

```csharp
// other codes...
return new ExternalAuthenticateResultModel
{
    ReturnUrl = returnUrl,
};

// other codes...

return new ExternalAuthenticateResultModel
{
    WaitingForActivation = true
};

// other codes...

return new ExternalAuthenticateResultModel();
```

### Add HttpOnly cookies to `Authenticate` method

Instead of directly providing the access token and refresh token in the response, we should include them as HttpOnly cookies in the response.

Find the `Authenticate` method and add tokens to cookies after creating tokens. Your code should look like this:

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
    TwoFactorRememberClientToken = twoFactorRememberClientToken,
    UserId = loginResult.User.Id,
    ReturnUrl = returnUrl
};
```

Add tokens to cookies like what was done upper, for the following methods.

* `RefreshToken`
* `ImpersonatedAuthenticate`
* `DelegatedImpersonatedAuthenticate`
* `LinkedAccountAuthenticate`
* `ExternalAuthenticate` 

### Create `IsRefreshTokenAvailable` Method

We must introduce to a new method to verify the availability of a refresh token. In your `TokenAuthController.cs` file, create a new method called `IsRefreshTokenAvailable` and insert the subsequent code:

```csharp
[HttpGet]
public bool IsRefreshTokenAvailable()
{
    var refreshToken = Request.Cookies["Abp.AuthRefreshToken"];
    return !string.IsNullOrWhiteSpace(refreshToken);
}
```

### Delete HttpOnly cookies in `Logout` method

To delete HttpOnly cookies, we need to configure the `Logout` method. In your `TokenAuthController.cs` file, find the `Logout` method and add the following code:

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

### Create `TenantIdCookieController.cs`

Create a new controller for setting tenant id cookie. Create a new controller named `TenantIdCookieController` in your `*.Web.Core` project then add the following code:

```csharp
using System;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;

namespace AngularHttpOnlyCookieDemo.Web.Controllers;

[Route("api/[controller]/[action]")]
public class TenantIdCookieController : AngularHttpOnlyCookieDemoControllerBase
{
    [HttpPost]
    public void SetTenantIdCookie(int tenantId)
    {
        if (tenantId == 0)
        {
            Response.Cookies.Delete("Abp.TenantId");
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

To read **HttpOnly** cookies in incoming requests, we will create a middleware. This middleware will read the HttpOnly cookies in incoming requests and append them to the request headers. This way, we will be able to access the HttpOnly cookies in our requests.

### Create `HttpOnlyMiddleware.cs`

Create a new middleware for adding **HttpOnly** cookies to the request headers. Create a new class named `HttpOnlyMiddleware` in your `*.Web.Host` project and add the following code:

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

Utilize the crafted middleware in your application by adding `app.UseHttpOnlyToken()` to the `Startup.cs` file in your `*.Web.Host` project as illustrated below:

```csharp
// Place before the Authentication middleware
app.UseHttpOnlyToken();

app.UseAuthentication();
```

## Step 3: Configure your `Angular` project

Due to the changes made on the backend, we need to adjust and remove certain parts of the project in your Angular application.

Run `npm run nswag` command to update `TokenAuthServiceProxy` service.

### Update `login.service.ts`

Update login method at `login.service.ts` and remove `accessToken` and `refreshToken` parameters. Your code should look like this:

```ts
private login(
    rememberMe?: boolean,
    twoFactorRememberClientToken?: string,
    redirectUrl?: string
): void {
    console.log('login reidrect');
    this.redirectToLoginResult(redirectUrl);
}
```

Then update `processAuthenticateResult` method at `login.service.ts` file as below:

```ts
private processAuthenticateResult(authenticateResult: AuthenticateResultModel, redirectUrl?: string) {
    this.authenticateResult = authenticateResult;

    if (authenticateResult.shouldResetPassword) {
        // Password reset

        this._router.navigate(['account/reset-password'], {
            queryParams: {
                c: authenticateResult.c,
            },
        });

        this.clear();
    } else if (authenticateResult.requiresTwoFactorVerification) {
        // Two factor authentication

        this._router.navigate(['account/send-code']);
    } else {
        // Successfully logged in

        if (authenticateResult.returnUrl && !redirectUrl) {
            redirectUrl = authenticateResult.returnUrl;
        }

        this.login(
            this.rememberMe,
            authenticateResult.twoFactorRememberClientToken,
            redirectUrl
        );
    }
}
```

### Update `tryAuthWithRefreshToken` method

Since cookies are http only, we can no longer access them via javascript, so we need to update this method.

Open `src/app/account/auth/zero-refresh-token.service.ts` and update `tryAuthWithRefreshToken` method as below.

```ts 
tryAuthWithRefreshToken(): Observable<boolean> {
    let refreshTokenObservable = new Subject<boolean>();

    this._tokenAuthService.isRefreshTokenAvailable().subscribe({
        next: (result) => {

            if (!result) {
                refreshTokenObservable.next(false);
                return;
            }

            this._tokenAuthService.refreshToken().subscribe({
                next: () => refreshTokenObservable.next(true),
                error: () => refreshTokenObservable.next(false)
            });
        }
    });

    return refreshTokenObservable;
}
```

### Comment cookie methods

Open `abp.js` file in your `src/assets/abp-web-resources` folder. 

Find the `abp.auth.setToken` and `abp.auth.getToken` functions, and comment out the function's body.

```js
abp.auth.getToken = function () {
    // Commented out for HttpOnly cookies
    // return abp.utils.getCookieValue(abp.auth.tokenCookieName);
}

abp.auth.setToken = function (authToken, expireDate) { 
    // Commented out for HttpOnly cookies
    // abp.utils.setCookieValue(abp.auth.tokenCookieName, authToken, expireDate, abp.appPath, abp.domain, { 'SameSite': abp.auth.tokenCookieSameSite });
};
```

Delete the places where the `abp.auth.setToken` function is used

* src/AppPreBootstrap.ts
* src/account/auth/zero-refresh-token.service.ts
* src/account/login/login.service.ts

Delete the places where the `abp.auth.getToken` function is used

*src/app/admin/users/users.component.ts*
```ts
setUsersProfilePictureUrl(users: UserListDto[]): void {
    for (let i = 0; i < users.length; i++) {
        let user = users[i];

        let profilePictureUrl =
            AppConsts.remoteServiceBaseUrl +
            '/Profile/GetProfilePictureByUser?userId=' +
            user.id;

        (user as any).profilePictureUrl = profilePictureUrl;

    }
}
```

*src/AppPreBootstrap.ts*
```js
private static getUserConfiguration(callback: () => void): any {
    return XmlHttpRequestHelper.ajax(
        'GET',
        AppConsts.remoteServiceBaseUrl + '/AbpUserConfiguration/GetAll',
        null,
        null,
        (response) => {
            let result = response.result;

            _merge(abp, result);

            abp.clock.provider = this.getCurrentClockProvider(result.clock.provider);

            AppPreBootstrap.configureLuxon();

            abp.event.trigger('abp.dynamicScriptsInitialized');

            AppConsts.recaptchaSiteKey = abp.setting.get('Recaptcha.SiteKey');
            AppConsts.subscriptionExpireNootifyDayCount = parseInt(
                abp.setting.get('App.TenantManagement.SubscriptionExpireNotifyDayCount')
            );

            DynamicResourcesHelper.loadResources(callback);
        }
    );
}
```

*src/app/shared/common/auth/app-auth.service.ts*
```ts
@Injectable()
export class AppAuthService {
    logout(reload?: boolean, returnUrl?: string): void {
        XmlHttpRequestHelper.ajax(
            'GET',
            AppConsts.remoteServiceBaseUrl + '/api/TokenAuth/LogOut',
            null,
            null,
            () => {
                this.logoutInternal(reload, returnUrl);
            },
            (errorResult: IAjaxResponse) => {
                if (errorResult.unAuthorizedRequest) {
                    abp.log.error(errorResult.error);
                    this.logoutInternal(reload, returnUrl);
                } else {
                    abp.message.error(errorResult.error.message);
                }
            }
        );
    }

    logoutInternal(reload?: boolean, returnUrl?: string): void {
        new LocalStorageService().removeItem(AppConsts.authorization.encrptedAuthTokenName, () => {
            if (reload !== false) {
                if (returnUrl) {
                    location.href = returnUrl;
                } else {
                    location.href = 'index.html';
                }
            }
        });
    }
}
```

Remove following lines from file uploaders in:

* src/app/admin/demo-ui-components/demo-ui-file-upload.component.ts
```ts	
event.xhr.setRequestHeader('Authorization', 'Bearer ' + abp.auth.getToken());
```

* src/app/shared/layout/profile/change-profile-picture-modal.component.ts
```ts	
event.xhr.setRequestHeader('Authorization', 'Bearer ' + abp.auth.getToken());
```

* src/app/admin/settings/tenant-settings.component.ts
```ts
uploaderOptions.authToken = 'Bearer ' + this._tokenService.getToken();
```

Find the `abp.auth.setRefreshToken` and comment the function's body.

```js
abp.auth.setRefreshToken = function (refreshToken, expireDate) { // abp.utils.setCookieValue(abp.auth.refreshTokenCookieName, refreshToken, expireDate, abp.appPath, abp.domain, { 'SameSite': abp.auth.tokenCookieSameSite });
};
```

Delete the places where the `abp.auth.setRefreshToken` function is used

*src/account/login/login.service.ts*
```ts
// remove this code
this._tokenService.setToken(accessToken, tokenExpireDate);

if (refreshToken && rememberMe) {
    let refreshTokenExpireDate = rememberMe
        ? new Date(new Date().getTime() + 1000 * refreshTokenExpireInSeconds)
        : undefined;
    this._tokenService.setRefreshToken(refreshToken, refreshTokenExpireDate);
}
```

### Update `setTenantIdCookie` Method

Update `setTenantIdCookie` method in `abp.js`. Do api call to `TenantIdCookieController` instead of setting cookie directly. Your code should look like this:

```js
abp.multiTenancy.setTenantIdCookie = function (tenantId) {
    return new Promise((resolve, reject) => {
        if (tenantId === undefined || tenantId === null) {
            tenantId = 0;
        }

        fetch(abp.appPath + 'api/TenantIdCookie/SetTenantIdCookie?tenantId=' + tenantId, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            }
        })
        .then(response => {
            if (!response.ok) {
                throw new Error('Network response was not ok');
            }
            return response.json();
        })
        .then(data => {
            console.log('Success:', data);
            resolve(data);
        })
        .catch(error => {
            console.error('Error:', error);
            reject(error);
        });
    });
    
};
```

As we are no longer able to handle cookies on the Angular side, switching tenants involves initiating an API call to the backend server. This API call facilitates the modification of cookies. Given our use of promise code in this API call, we have accordingly modified the `setTenantIdCookie` method to return a promise. Let's add definition of `setTenantIdCookie` method in `typings.d.ts` file.

*/src/typings.d.ts*
```ts
namespace multiTenancy{
    function setTenantIdCookie(tenantId?: number): Promise<void>;
}
```

Due to our utilization of **Promises**, find the usages of `setTenantIdCookie` in your project, and refactor them using the then and catch blocks as below:

*src/AppPreBootstrap.ts*
```ts
private static impersonatedAuthenticate(impersonationToken: string, tenantId: number, callback: () => void): void {
    abp.multiTenancy.setTenantIdCookie(tenantId).then(() => {

        let requestHeaders = AppPreBootstrap.getRequetHeadersWithDefaultValues();

        XmlHttpRequestHelper.ajax(
            'POST',
            AppConsts.remoteServiceBaseUrl +
            '/api/TokenAuth/ImpersonatedAuthenticate?impersonationToken=' +
            impersonationToken,
            requestHeaders,
            null,
            (response) => {
                let result = response.result;
                AppPreBootstrap.setEncryptedTokenCookie(result.encryptedAccessToken, () => {
                    callback();
                    location.search = '';
                });
            }
        );
    });
}
```

## Conclusion

That's it! Now, you can use **HttpOnly** cookies in your ASP.NET Zero Angular application.

![Angular http only cookie](images/angular-http-only-cookie.png)