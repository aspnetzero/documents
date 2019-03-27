# Running the Application

**It's finished**! We can test the application. Run the project,
**login** as the **host admin** (click Change link and clear tenancy
name) shown below:

<img src="images/login-as-host-3.png" alt="Login as host" class="img-thumbnail" style="width:100.0%" />

After login, we see the **tenant list** which only contains a
**default** tenant. We can create a new tenant:

<img src="images/create-tenant-6.png" alt="Creating tenant" class="img-thumbnail" />

I created a new tenant named **trio**. Now, tenant list has two tenants:

<img src="images/tenant-management-phonebook-2.png" alt="Tenant management page" class="img-thumbnail" style="width:100.0%" />

 I can **logout** and login as **trio** tenant **admin** (Change current
tenant to trio):

<img src="images/login-as-trio1.png" alt="Login as tenant admin" class="img-thumbnail" style="width:100.0%" />

After login, we see that phone book is empty:

<img src="images/phonebook-empty1.png" alt="Empty phonebook of new tenant" class="img-thumbnail" />

It's empty because trio tenant has a completely isolated people list.
You can add people here, logout and login as different tenants (you can
login as default tenant for example). You will see that each tenant has
an isolated phone book and can not see other's people.

## Next

- [Conclusion](Developing-Step-By-Step-Angular-Conclusion)