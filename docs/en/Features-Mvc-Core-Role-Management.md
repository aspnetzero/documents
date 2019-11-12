#### Role Management

When we click Administration/Roles menu, we enter the role management page:

<img src="images/role-management-core-3.png" alt="Role management page" class="img-thumbnail" />

Roles are used to **group permissions**. When a user has a role, then he/she will have all permissions of that role.

A role is represented by the **Role** class. Role class can be extended by adding new properties.

**RoleManager** performs domain logic, **RoleAppService** performs application logic for roles.

Roles can be dynamic or static:

- **Static role**: A static role has a known **name** (like 'admin') and can not change this name (we can change **display name**). It's exists on the system startup and can not be deleted. Thus, we can write our code based on a static role name.
- **Dynamic role**: We can create a dynamic role after deployment. Then we can grant permissions for that role, we can assign the role to some users and we can delete it. We can not know the names of dynamic roles in development time.

One or more roles can be set as **default**. Default roles are assigned to new added/registered users as default. This is not a development time property and can be set or changed after deployment.

In startup project, we have static **admin** role for host (for multi-tenant apps). Also, we have static **admin** and **user** roles for tenants. **Admin** roles have all permissions granted by default. **User** role is the **default** role for new users and has no permission by default. These can be changed easily. See **StaticRoleNames** class for all static roles and **AppRoleConfig** for changing static roles.

##### Role Permissions

Since roles are used to group permissions, we can set permissions of a role while creating or editing as shown below:

<img src="images/role-permissions-core-1.png" alt="Role Permissions" class="img-thumbnail" />

*Note that not all permissions shown in the figure above*

Every tenant has its own roles and any change in roles for a tenant does not affect other tenants. Also, host has also own isolated roles.

## Next

- [User Management](Features-Mvc-Core-User-Management)


