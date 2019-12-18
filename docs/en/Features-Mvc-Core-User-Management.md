# User Management

When we click Administration/Users menu, we enter the user management page:

<img src="images/user-management-core-3.png" alt="User management" class="img-thumbnail" />

**Users** are people who can **login** to the application and perform some operations based on their **permissions**.

**User** class represents a user. User class [can be extended](Extending-Existing-Entities.md) by adding new properties.

**UserManager** is used to perform domain logic, **UserAppService** is used to perform application logic for users.

A user can have zero or more **roles**. If a user has more than one role, he inherits union of permissions of all these roles. Also, we can set **user-specific permission**. A user specific permission setting overrides role settings for this permission. A screenshot of user permission dialog:

<img src="images/user-permissions-1.png" alt="User Permissions" class="img-thumbnail" />

*Note that not all permissions shown in the figure above*

A dialog is used to create/edit a user:

<img src="images/edit-user-3.png" alt="Editing User" class="img-thumbnail" />

We can change user's **password**, make her **active/passive** and so on... A user can have a **profile picture**. It can be changed by the user (See User Menu section). **Admin** user can not be deleted as a business rule. If you don't want to use admin, you can make it passive.

### Excel operations

User list can be downloaded as an Excel file. Also, new users can be imported from an Excel file. In order to import users from an Excel file, ASP.NET Zero requires a specific excel file format. You can download a sample import template on the user list by clicking the "click here" link under the "Excel operations" dropdown button.

<img src="D:/github/documents/docs/en/images/user-list-excel-operations-import-template-link.png" alt="Sample Excel import template link" class="img-thumbnail" />

When you import users from Excel, if there are errors while importing users to ASP.NET Zero's database, invalid users will be saved to an Excel file with the reason (validation error message or exception message) and the user who made the import operation will be notified via ASP.NET Zero's notification system. So, the user can click the notification and see the result Excel file, fix the validation errors and import the failed users again.

## User Impersonation

As admin (or any allowed user), we may want to login as a user and perform operations on behalf of that user, without knowing his password. When we click "**Login as this user**" icon in the actions of a user, we
are automatically redirected and logged in as this user. This is called as "**user impersonation**". When we impersonate a user, a "**back to my account**" option is added to the user profile menu:

<img src="images/back-to-my-account-link-3.png" alt="Back to my account link" class="img-thumbnail" />

In an impersonated account, we can only perform operations allowed to that user. That means, everything **exactly** works as same as this user logged in himself. The only difference is shown in audit logs which
indicates that operations are performed by somebody else. Notice that; Also a **red 'back' icon** shown near to the user name to indicate that you are in an impersonated account.

Impersonation is done in **AccountController** of the Web project.

## Next

- [Language Management](Features-Mvc-Core-Language-Management)


