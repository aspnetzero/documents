## Sign In Without Specifying Tenant

Normally, **ASP.NET Zero** uses tenant information during the login process. This document shows you how to implement the login process without tenant information. 

***Important Note:*** *Your user's email addresses have to be unique to implement this solution. Otherwise, this solution may not work correctly.*

#### Updating LogInManager

* First of all, open `LogInManager`. *(It is located in **aspnet-core\src\\[YOURAPPNAME].Application\Authorization** folder.)*

* Add lines shown below

  ``````csharp
  UserStore _userStore
  public LogInManager(
  	//....
      UserStore userStore
  ){
      _userStore = userStore;
  }
  
  [UnitOfWork]
  public async Task<AbpLoginResult<Tenant, User>> LoginAsync(UserLoginInfo login)
  {
      var result = await LoginAsyncInternal(login);
      await SaveLoginAttemptAsync(result, result.Tenant.Name, login.ProviderKey + "@" + login.LoginProvider);
      return result;
  }
  
  protected async Task<AbpLoginResult<Tenant, User>> LoginAsyncInternal(UserLoginInfo login)
  {
      if (login == null || login.LoginProvider.IsNullOrEmpty() || login.ProviderKey.IsNullOrEmpty())
      {
          throw new ArgumentException("login");
      }
      using (UnitOfWorkManager.Current.DisableFilter(AbpDataFilters.MayHaveTenant))
      {
          var user = await _userStore.FindAsync(login);
          if (user == null)
          {
              return new AbpLoginResult<Tenant, User>(AbpLoginResultType.UnknownExternalLogin);
          }
          //Get and check tenant
          Tenant tenant = null;
          if (!MultiTenancyConfig.IsEnabled)
          {
              tenant = await GetDefaultTenantAsync();
          }
          else if (user.TenantId.HasValue)
          {
              tenant = await TenantRepository.FirstOrDefaultAsync(t => t.Id == user.TenantId);
              if (tenant == null)
              {
                  return new AbpLoginResult<Tenant, User>(AbpLoginResultType.InvalidTenancyName);
              }
              if (!tenant.IsActive)
              {
                  return new AbpLoginResult<Tenant, User>(AbpLoginResultType.TenantIsNotActive, tenant);
              }
          }
          return await CreateLoginResultAsync(user, tenant);
      }
  }
  ``````

  Then, your `LogInManager` will be able to use given user's tenant for login process.
  
  
  
  #### Updating UserManager
  
* Go to `UserManager`. *(It is located in **aspnet-core\src\\[YOURAPPNAME].Core\Authorization\Users** folder.)*

* And add following lines;

```csharp
public async Task<int?> TryGetTenantIdOfUser(string userEmail)
  {
      using (_unitOfWorkManager.Current.DisableFilter(AbpDataFilters.MayHaveTenant))
      {
          var user = await Users.SingleOrDefaultAsync(u => u.EmailAddress == userEmail.Trim());
          return user?.TenantId;
      }
  }
  ```
  
  
  
  #### Updating TokenAuthController
  
* Then, go to  `TokenAuthController`. *(It is located in **aspnet-core\src\[YOURAPPNAME].Web.Core\Controllers** folder.)*
  
* Replace the function named `GetTenancyNameOrNull` with the following content
  
  ```csharp
  private async Task<string> GetTenancyNameOrNull(string email)
  {
      var tenantId = await _userManager.TryGetTenantIdOfUser(email);
      if (!tenantId.HasValue)
      {
          return null;
      }
      return _tenantCache.GetOrNull(tenantId.Value)?.TenancyName;
  }
  ```

* Replace the function named `Authenticate([FromBody] AuthenticateModel model) ` with the following content

  ```csharp
  [HttpPost]
  public async Task<AuthenticateResultModel> Authenticate([FromBody] AuthenticateModel model)
  {
      if (UseCaptchaOnLogin())
      {
          await ValidateReCaptcha(model.CaptchaResponse);
      }
      
      var loginResult = await GetLoginResultAsync(
          model.UserNameOrEmailAddress,
          model.Password,
          await GetTenancyNameOrNull(model.UserNameOrEmailAddress)//use new GetTenancyNameOrNull method that you add previously
      );
      
      var returnUrl = model.ReturnUrl;
      if (model.SingleSignIn.HasValue && model.SingleSignIn.Value &&
          loginResult.Result == AbpLoginResultType.Success)
      {
          loginResult.User.SetSignInToken();
          returnUrl = AddSingleSignInParametersToReturnUrl(model.ReturnUrl, loginResult.User.SignInToken,
              loginResult.User.Id, loginResult.User.TenantId);
      }
      
      //Password reset
      if (loginResult.User.ShouldChangePasswordOnNextLogin)
      {
          loginResult.User.SetNewPasswordResetCode();
          return new AuthenticateResultModel
          {
              ShouldResetPassword = true,
              PasswordResetCode = loginResult.User.PasswordResetCode,
              UserId = loginResult.User.Id,
              ReturnUrl = returnUrl
          };
      }
      
      //Two factor auth
      await _userManager.InitializeOptionsAsync(loginResult.Tenant?.Id);
      
      string twoFactorRememberClientToken = null;
      if (await IsTwoFactorAuthRequiredAsync(loginResult, model))
      {
          if (model.TwoFactorVerificationCode.IsNullOrEmpty())
          {
              //Add a cache item which will be checked in SendTwoFactorAuthCode to prevent sending unwanted two factor code to users.
              _cacheManager
                  .GetTwoFactorCodeCache()
                  .Set(
                      loginResult.User.ToUserIdentifier().ToString(),
                      new TwoFactorCodeCacheItem()
                  );
              return new AuthenticateResultModel
              {
                  RequiresTwoFactorVerification = true,
                  UserId = loginResult.User.Id,
                  TwoFactorAuthProviders = await _userManager.GetValidTwoFactorProvidersAsync(loginResult.User),
                  ReturnUrl = returnUrl
              };
          }
          twoFactorRememberClientToken = await TwoFactorAuthenticateAsync(loginResult.User, model);
      }
      
      // One Concurrent Login 
      if (AllowOneConcurrentLoginPerUser())
      {
          await _userManager.UpdateSecurityStampAsync(loginResult.User);
          await _securityStampHandler.SetSecurityStampCacheItem(loginResult.User.TenantId, loginResult.User.Id,
              loginResult.User.SecurityStamp);
          loginResult.Identity.ReplaceClaim(new Claim(AppConsts.SecurityStampKey,
              loginResult.User.SecurityStamp));
      }
      
      var refreshToken = CreateRefreshToken(await CreateJwtClaims(loginResult.Identity, loginResult.User,
          tokenType: TokenType.RefreshToken));
      
      var accessToken = CreateAccessToken(await CreateJwtClaims(loginResult.Identity, loginResult.User,
          refreshTokenKey: refreshToken.key));
      
      return new AuthenticateResultModel
      {
          AccessToken = accessToken,
          ExpireInSeconds = (int) _configuration.AccessTokenExpiration.TotalSeconds,
          RefreshToken = refreshToken.token,
          RefreshTokenExpireInSeconds = (int) _configuration.RefreshTokenExpiration.TotalSeconds,
          EncryptedAccessToken = GetEncryptedAccessToken(accessToken),
          TwoFactorRememberClientToken = twoFactorRememberClientToken,
          UserId = loginResult.User.Id,
          ReturnUrl = returnUrl
      };
  }
  ```

* Replace the function named `ExternalAuthenticate([FromBody] ExternalAuthenticateModel model) ` with the following content

  ```csharp
  [HttpPost]
  public async Task<ExternalAuthenticateResultModel> ExternalAuthenticate([FromBody] ExternalAuthenticateModel model)
  {
      var externalUser = await GetExternalUserInfo(model);
      var loginResult = await _logInManager.LoginAsync(new UserLoginInfo(model.AuthProvider, model.ProviderKey, model.AuthProvider));
      switch (loginResult.Result)
      {
          case AbpLoginResultType.Success:
          {
              var refreshToken = CreateRefreshToken(await CreateJwtClaims(loginResult.Identity, loginResult.User,
                  tokenType: TokenType.RefreshToken));
              var accessToken = CreateAccessToken(await CreateJwtClaims(loginResult.Identity, loginResult.User,
                  refreshTokenKey: refreshToken.key));
              var returnUrl = model.ReturnUrl;
              
              if (model.SingleSignIn.HasValue && model.SingleSignIn.Value &&
                  loginResult.Result == AbpLoginResultType.Success)
              {
                  loginResult.User.SetSignInToken();
                  returnUrl = AddSingleSignInParametersToReturnUrl(model.ReturnUrl, loginResult.User.SignInToken,
                      loginResult.User.Id, loginResult.User.TenantId);
              }
              
              return new ExternalAuthenticateResultModel
              {
                  AccessToken = accessToken,
                  EncryptedAccessToken = GetEncryptedAccessToken(accessToken),
                  ExpireInSeconds = (int) _configuration.AccessTokenExpiration.TotalSeconds,
                  ReturnUrl = returnUrl,
                  RefreshToken = refreshToken.token,
                  RefreshTokenExpireInSeconds = (int) _configuration.RefreshTokenExpiration.TotalSeconds
              };
          }
          case AbpLoginResultType.UnknownExternalLogin:
          {
              var newUser = await RegisterExternalUserAsync(externalUser);
              if (!newUser.IsActive)
              {
                  return new ExternalAuthenticateResultModel
                  {
                      WaitingForActivation = true
                  };
              }
              //Try to login again with newly registered user!
              loginResult = await _logInManager.LoginAsync(
                  new UserLoginInfo(model.AuthProvider, model.ProviderKey, model.AuthProvider)
              );
              
              if (loginResult.Result != AbpLoginResultType.Success)
              {
                  throw _abpLoginResultTypeHelper.CreateExceptionForFailedLoginAttempt(
                      loginResult.Result,
                      model.ProviderKey,
                      loginResult?.Tenant?.Name
                  );
              }
              
              var refreshToken = CreateRefreshToken(await CreateJwtClaims(loginResult.Identity,
                  loginResult.User, tokenType: TokenType.RefreshToken)
              );
              
              var accessToken = CreateAccessToken(await CreateJwtClaims(loginResult.Identity,
                  loginResult.User, refreshTokenKey: refreshToken.key));
              
              return new ExternalAuthenticateResultModel
              {
                  AccessToken = accessToken,
                  EncryptedAccessToken = GetEncryptedAccessToken(accessToken),
                  ExpireInSeconds = (int) _configuration.AccessTokenExpiration.TotalSeconds,
                  RefreshToken = refreshToken.token,
                  RefreshTokenExpireInSeconds = (int) _configuration.RefreshTokenExpiration.TotalSeconds
              };
          }
          default:
          {
              throw _abpLoginResultTypeHelper.CreateExceptionForFailedLoginAttempt(
                  loginResult.Result,
                  model.ProviderKey,
                  loginResult?.Tenant?.Name
              );
          }
      }
  }
  ```

Then your project will be able to use without specifying tenant.

#### More

For a more stable UI, you can remove the tenant selection model used for login operations.

Go to **aspnet-zero-core\angular\src\account\account.component.ts** and add add **login** to **tenantChangeDisabledRoutes**

```typescript
 tenantChangeDisabledRoutes: string[] = [
        'select-edition',
        'buy',
        'upgrade',
        'extend',
        'register-tenant'
     	//...
        'login'//add login
    ];
```

<img src="images/login-page-with-tenant-change.png" class="img-thumbnail" />

<img src="images/login-page-without-tenant-change.png" class="img-thumbnail" />
