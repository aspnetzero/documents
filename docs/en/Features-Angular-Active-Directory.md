# Active Directory (LDAP)

LDAP (Active Directory) Authentication is disabled by default.

To enable LDAP on server-side open `*CoreModule.cs` in ***.Core** project, uncomment the following line

```csharp
Configuration.Modules.ZeroLdap().Enable(typeof(AppLdapAuthenticationSource));
```

See [server side](Features-Mvc-Core-Tenant-Active-Directory) to enable LDAP. Once we enable, we can see **LDAP settings** section in the settings page:

<img src="images/tenant-settings-ldap-1.png" alt="LDAP Settings" class="img-thumbnail" />

We can check "**Enable LDAP Authentication**" to enable it. If the server works in domain and application runs with a domain user or local system, then generally even **no need** to set Domain name, user and password. You can logout and then login with your **domain user name and password**. If not, you should set these credentials.

## Next

- [Maintenance](Features-Angular-Maintenance)