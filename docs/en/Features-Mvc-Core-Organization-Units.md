# Organization Units

Organization units (OU) are used to hierarchically group user and entities. Then you can get user or entities based on their OUs. When we click Administration/Organization units, we enter the related page:

<img src="images/organization-units-page-core-3.png" alt="Organization units page" class="img-thumbnail" />

Here, we can manage OUs (create, edit, delele, move) and members (add/remove).

**OrganizationUnitManager** is used to manage OUs, **UserManager** is used to manage OU members in the code. **OrganizationUnitAppService** performs the application logic.

In the left OU tree, we can **right click** to an OU (or left click to **arrow** at the right) to open **context menu** for OU operations. When we try to **add a member**, a modal is shown to select the user:

<img src="images/select-user-lookup-core-3.png" alt="Select a user dialog" class="img-thumbnail" />

This is actually a **generic lookup modal** and can be used to select any type of entity (see
`Areas\App\Views\Common\Modals\_LookupModal.cshtml` and it's related script file). To select the users, we created **FindUsers** method in **CommonLookupAppService** then configured the modal to work with this method (see `view-resources/Areas/App/Views/OrganizationUnits/index.js` file for usage of lookupModal.open).

See [organization unit management document](https://aspnetboilerplate.com/Pages/Documents/Zero/Organization-Units) for more information.

## Next

- [Role Management](Features-Mvc-Core-Role-Management)