# Role Management

When we click Administration / Roles menu, we enter to the role management page:

<img src="images/role-management-core-3.png" alt="Role management page" class="img-thumbnail" />

Roles are used to **group permissions**. When a user has a role, then the user will have all permissions of that role.

A role is represented by the **Role** class. Role class [can be extended](Extending-Existing-Entities.md) by adding new properties.

Roles can be dynamic or static:

- **Static role**: A static role has a known **name** (like 'admin') and this name can't be changed. (But the **display name** can be changed). It's created on the system startup and can not be deleted on the UI. Thus, we can write codes based on a static role name.
- **Dynamic role**: We can create a dynamic role after deployment. Then we can grant permissions for that role, we can assign the role to some users and we can delete it. We can not know names of dynamic roles in development time.

One or more roles can be set as **default**. Default roles are assigned to newly added/registered users by default. This is not a development time property and can be set or changed after deployment.

In startup project, we have static **admin** role for host (for multi-tenant apps). Also, we have static **admin** and **user** roles for tenants. **Admin** roles have all permissions granted by default.
**User** role is the **default** role for new users and has no permission by default. These can be changed easily in the **AppRoleConfig.cs** class under "**/Authorization/Roles/**" folder in the *.Core project. 

Static roles are seeded in **HostRoleAndUserCreator.cs** and **TenantRoleAndUserBuilder.cs** classes under "**Migrations/Seed/**" folder in *.EntityFrameworkCore project.

#### Role Permissions

Since roles are used to group permissions, we can set permissions of a role while creating or editing as shown below:

<img src="images/role-permissions-core-1.png" alt="Role Permissions" class="img-thumbnail" />

*Note that not all permissions shown in the figure above.*

Every tenant has it's **own roles** and any change in roles for a tenant does not effect other tenants. Also, host has also it's own isolated roles.

#### Resetting Role Permissions

Permissions of a static role can be changed like the permissions of any other role, and after a while it may be hard to tell which permissions the role had at the beginning. To go back, open the static role and click the **Reset to default permissions** button in the edit role modal. After you confirm, all permission changes made for that role are removed and the role gets its default permissions back. The change is saved immediately and affects every user in the role.

The default permissions of a static role come from its definition in the **AppRoleConfig** class. A static role defined with `grantAllPermissionsByDefault: true` gets all permissions back, and any other static role gets the permissions in its `GrantedPermissions` list. In the startup project, this means the **admin** roles get all permissions back and the **user** role gets no permission back. The button is not shown for dynamic roles, since they have no default permissions to go back to.

If you want a static role to start with some permissions, add them to its definition in **AppRoleConfig**:

```csharp
roleManagementConfig.StaticRoles.Add(
    new StaticRoleDefinition(
        StaticRoleNames.Tenants.User,
        MultiTenancySides.Tenant)
    {
        GrantedPermissions =
        {
            AppPermissions.Pages_DemoUiComponents
        }
    }
);
```

Permissions granted to a role from code, for example in **TenantRoleAndUserBuilder**, are stored like the changes made on the UI, so resetting the role removes them as well. Define default permissions in **AppRoleConfig** so that they survive a reset.

On the server side, the reset is done by the `ResetRolePermissions` method of **RoleAppService**, which calls `RoleManager.ResetPermissionsToDefaultAsync`.

## Next

- [User Management](Features-Angular-User-Management)