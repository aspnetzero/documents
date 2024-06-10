# Switching Between Organization Units

In ASP.NET Zero projects, the ability for users to switch between different organizational units is essential, especially for multi unit organizations.Allowing users to select and change their active unit during their sessions enhances both the user experience and operational efficiency. However, managing this transition correctly and securely can often be complex.

## Implementing of Organization Unit Switching Between

In this section, we'll delve into the implementation details of enabling users to switch between different organizational units within an ASP.NET Zero project. 

### Creating Claims Principal for Organization Unit

In this section, we create this functionality by overriding the **CreateAsync** function within the `UserClaimsPrincipalFactory` class. This modification ensures that we can access the organizational unit information for logged-in users.

```csharp
public override async Task<ClaimsPrincipal> CreateAsync(User user)
{
    var userWithOrgUnits = await GetUserWithOrganizationUnitsAsync(user.Id);

    var claim = await base.CreateAsync(userWithOrgUnits);

    string organizationUnitsJson = JsonSerializer.Serialize(userWithOrgUnits.OrganizationUnits.First());

    claim.Identities.First().AddClaim(new Claim("Organization_Unit", organizationUnitsJson));

    return claim;
}
```

The first code snippet overrides the **CreateAsync** method to incorporate organization unit information into the claims principal. 

Within this context, we add the first element of the Organization Unit as a claim during user principal creation.

```csharp
private async Task<User> GetUserWithOrganizationUnitsAsync(long userId)
{
    return await UserManager.Users
        .Include(u => u.OrganizationUnits)
        .Where(x => x.Id == userId)
        .FirstOrDefaultAsync();
}
```

The second snippet demonstrates the asynchronous method GetUserWithOrganizationUnitsAsync, which retrieves a user with their associated organization units from the database.

### Modifying the Organization Unit in Session

Here, we implement functionality to modify the organizational unit within the session, following the guidelines outlined in this [document](https://aspnetboilerplate.com/Pages/Documents/Articles%5CHow-To%5Cadd-custom-session-field-aspnet-core).

```csharp
public class MyAppSession : ClaimsAbpSession, ITransientDependency
{
    private const string OrganizationUnitClaimTypeName = "Organization_Unit";
    private readonly UserManager _userManager;
    public MyAppSession(
        IPrincipalAccessor principalAccessor,
        IMultiTenancyConfig multiTenancy,
        ITenantResolver tenantResolver,
        IAmbientScopeProvider<SessionOverride> sessionOverrideScopeProvider,
        UserManager userManager) :
        base(principalAccessor, multiTenancy, tenantResolver, sessionOverrideScopeProvider)
    {
        _userManager = userManager;
    }

    public string OrganizationUnit => PrincipalAccessor.Principal?.Claims.FirstOrDefault(c => c.Type == OrganizationUnitClaimTypeName)?.Value;

    public async Task SwitchOrganizationUnitAsync(UserOrganizationUnit newUserOrganizationUnit)
    {
        var oldOrganizationUnit = JsonSerializer.Deserialize<UserOrganizationUnit>(OrganizationUnit);

        SetOrganizationUnit(newUserOrganizationUnit);

        await _userManager.RemoveFromOrganizationUnitAsync(oldOrganizationUnit.UserId, oldOrganizationUnit.OrganizationUnitId);
    }

    private void SetOrganizationUnit(string newOrganizationUnit)
    {
        var identity = PrincipalAccessor.Principal?.Identity as ClaimsIdentity;
        if (identity != null)
        {
            var oldOrganizationUnitClaim = identity.FindFirst(OrganizationUnitClaimTypeName);
            if (oldOrganizationUnitClaim != null)
            {
                identity.RemoveClaim(oldOrganizationUnitClaim);
            }

            identity.AddClaim(new Claim(OrganizationUnitClaimTypeName, newOrganizationUnit));
        }
    }
}
```

With these additions, the `MyAppSession` class now includes methods to both retrieve and set the organization unit information in the session. This enables management of organization unit data throughout the user's session.

### Usage Example: Switching Organization Units

In this section, we will demonstrate how to use a method for switching a user's organization unit within a new service class. The following example shows the `OrganizationUnitManager` class, which uses `MyAppSession` to allow a user to be removed from their current organization unit and added to a new one.

```csharp
public class OrganizationUnitManager
{
    private readonly MyAppSession _myAppSession;
    private readonly UserManager _userManager;

    public OrganizationUnitManager(MyAppSession myAppSession, UserManager userManager)
    {
        _myAppSession = myAppSession;
        _userManager = userManager;
    }

    public async Task SwitchOrganizationUnitAsync(UserOrganizationUnit newUserOrganizationUnit)
    {
        var oldOrganizationUnit = JsonSerializer.Deserialize<UserOrganizationUnit>(_myAppSession.OrganizationUnit);

        _myAppSession.SetOrganizationUnit(newUserOrganizationUnit);

        if (oldOrganizationUnit != null)
        {
            await _userManager.RemoveFromOrganizationUnitAsync(oldOrganizationUnit.UserId, oldOrganizationUnit.OrganizationUnitId);
        }

        await _userManager.AddToOrganizationUnitAsync(newUserOrganizationUnit.UserId, newUserOrganizationUnit.OrganizationUnitId);
    }
}
```

#### Usage Steps

1. **Retrieve the Current Organization Unit:** The oldOrganizationUnit is deserialized from the current session's OrganizationUnit claim.
2. **Set the New Organization Unit:** The SetOrganizationUnit method of MyAppSession is called to set the new organization unit in the session.
3. **Remove from the Old Unit:** The user is removed from the old organization unit using _userManager.RemoveFromOrganizationUnitAsync.
4. **Add to the New Unit:** The user is added to the new organization unit using _userManager.AddToOrganizationUnitAsync.


## Conclusion

Switching between organizational units in ASP.NET Zero projects can increase your organization's flexibility in transitions between organizations and facilitate organizational management. With a custom session, it is possible to switch between the user's current organization information and the organization. This work can also simplify organizational operational processes.