## Enabling LDAP (Active Directory) Authentication

LDAP (Active Directory) Authentication is disabled by default. In CoreModule class in .Core project, we should enable the following line:

```
Configuration.Modules.ZeroLdap().Enable(typeof(AppLdapAuthenticationSource));
```

Then, we can see **LDAP settings** section in settings page:

<img src="images/tenant-settings-ldap-1.png" alt="LDAP Settings" class="img-thumbnail" />

We can check "**Enable LDAP Authentication**" to enable it. If the server works in domain and application runs with a domain user or local system, then generally even **no need** to set Domain name, user and
password. You can logout and then login with your **domain user name and password**. If not, you should set these credentials.

**Note:** Password is stored encrypted after version v9.0

## Next

- [Maintenance](Features-Mvc-Core-Maintenance)