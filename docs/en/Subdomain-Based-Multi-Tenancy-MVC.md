# Subdomain Based Multi Tenancy in ASP.NET Zero (MVC)

## Introduction

This document provides detailed guidance for configuring and troubleshooting subdomain based multi tenancy in ASP.NET Zero projects utilizing an MVC frontend. It expands upon the foundational knowledge offered in the official documentation, aiming to cover advanced configurations, development environment strategies, and common issues.

This document assumes you have a basic understanding of ASP.NET Zero's multi tenancy concepts and have reviewed the main [Overview MVC](Overview-Angular.md) documentation.

## 1. Configuration for Subdomain Based Tenancy

This section involves SSL management, web server setup, and specific MVC application settings.

### SSL Certificate Management for Wildcard Domains

* **Requirement:** For HTTPS, a wildcard SSL certificate (e.g., for `*.mydomain.com`) or a Subject Alternative Name (SAN) certificate covering all required tenant subdomains is essential. This secures communication for all your tenants.
* **Setup:**
    * Obtain a wildcard or appropriate SAN certificate from a Certificate Authority (CA).
    * Install the certificate on your web server (e.g., IIS, Nginx, Apache) or load balancer.
    * For IIS, this involves binding the HTTPS protocol (port 443) to the installed wildcard certificate for your application's site.
* **Note on `WebSiteRootAddress`:** If your `WebSiteRootAddress` for tenants (e.g., `https://{TENANCY_NAME}.mydomain.com/`) uses a different primary domain or distinct sets of subdomains, ensure your SSL certificate(s) cover all relevant wildcard domains.

### Web Server Detailed Configuration (Example: IIS)

* **Host Headers & Bindings:**
    * Configure a single website in IIS to respond to requests for all tenant subdomains.
    * For HTTP binding (port 80), you can often leave the "Host Name" field empty if it's the only site on that IP, or specify each primary domain if needed.
    * For HTTPS binding (port 443), select the appropriate wildcard SSL certificate. The host name field might be left blank or set if your IIS version supports specific SNI configurations with wildcards, but often the wildcard nature is primarily handled by the certificate itself. The key is that IIS listens for traffic on the IP and port, and the certificate validates for `*.mydomain.com`.
* **Default Document/Application:** Ensure your IIS site is configured to correctly serve the MVC application (which handles routing to appropriate views and static assets) for all subdomain requests.
* **URL Rewrite (If Necessary):** While ASP.NET Zero handles tenant resolution based on the host, complex infrastructures involving reverse proxies might require URL Rewrite rules. However, for standard deployments, this is generally not needed for the core multi tenancy functionality.

### MVC Application Considerations

* **`WebSiteRootAddress` Configuration:**
    * It is critical that the `{TENANCY_NAME}` placeholder is used in the `App:WebSiteRootAddress` setting within your `YourProjectName.Web.Mvc/appsettings.json` file.
    * Example (in `appsettings.json`):
        ```json
        {
          "App": {
            "WebSiteRootAddress": "https://{TENANCY_NAME}[.mydomain.com/](https://.mydomain.com/)"
          }
          //... Other Settings
        }
        ```
* **URL Generation in Views/Controllers:**
    * ASP.NET Core's URL helpers (e.g., `@Url.Action()`, `<a asp-controller="..." asp-action="...">` tag helpers, `RedirectToAction`) will typically generate URLs relative to the current request's host, which includes the tenant subdomain. This is because the routing system and URL generation are aware of the incoming request's scheme and host.
    * This ensures that links within the application correctly point to tenant specific URLs without manual string manipulation of the domain in most cases.
* **Static Assets:** Static assets (CSS, JS, images) served by the MVC application (typically from `wwwroot`) will also be accessed via the tenant specific subdomain, as the browser requests them relative to the current page's domain.

## 2. Development Environment Strategies for Subdomain Testing

Testing subdomain based multi tenancy locally requires a few extra steps compared to path based tenancy or using the tenant switch dialog.

### Using the `hosts` File

* **Purpose:** To simulate DNS resolution for your subdomains on your local machine, you can edit the `hosts` file.
    * Windows: `C:\Windows\System32\drivers\etc\hosts`
    * Linux/macOS: `/etc/hosts`
* **Example:** Add entries to map your desired local subdomains to your loopback address (`127.0.0.1`):
    ```
    127.0.0.1 tenant1.localhost
    127.0.0.1 tenant2.localhost
    ```
* **Accessing the App:** After saving the `hosts` file (you might need administrator privileges), you can access your MVC application in the browser using URLs like `http://tenant1.localhost:44302` (or your configured MVC development port, e.g., from `launchSettings.json`).

### Development Web Server Configuration (ASP.NET Core MVC)

* **`launchSettings.json`:**
    * Configure the `applicationUrl` in your `YourProjectName.Web.Mvc/Properties/launchSettings.json` file to include the hostnames you've defined in your `hosts` file.
    * When you run the project, Kestrel (or IIS Express if configured) will listen on these specified URLs. You can then browse to `http://tenant1.localhost:44302` or `https://tenant1.localhost:44302`.

### Local SSL with Subdomains

* Standard `localhost` development certificates provided by `dotnet dev-certs https` will not be valid for custom local subdomains like `tenant1.localhost`. This will result in browser SSL warnings.
* **Solutions:**
    1.  **HTTP Locally:** For simplicity, you can develop and test the subdomain logic using HTTP locally. Add HTTP versions of your tenant localhost URLs to `launchSettings.json`.
    2.  **`mkcert` Tool:** Use a tool like `mkcert` to create a locally trusted Certificate Authority (CA) and then generate wildcard certificates (e.g., `*.localhost`). You would then configure Kestrel to use these certificates. This provides a more accurate simulation of a production HTTPS environment.
    3.  **Trust Self Signed Certificate (IIS Express):** If using IIS Express and it generates a certificate for `tenant1.localhost`, you might still need to manually trust it in your browser or system.

### Interaction with Tenant Switch Dialog

* When subdomains are correctly configured and resolved (even in a local development environment using the `hosts` file), ASP.NET Zero should automatically identify the tenant from the URL's subdomain.
* In such cases, the manual 'Tenant Switch' dialog becomes less relevant for requests made to these subdomain URLs, as the tenant context is already established.

## 3. Tenant Resolution Flow Clarification

A clear understanding of the tenant resolution process is essential to ensure accurate tenant identification and proper request routing in a multi tenant architecture. This step determines which tenant context should be applied based on the incoming request, typically using elements like the subdomain, header, or query string.

### Backend (ASP.NET Core)

1.  An incoming HTTP request arrives at the ASP.NET Core MVC application.
2.  ASP.NET Zero's middleware, specifically implementations of `ITenantResolveContributor` (like `DomainTenantResolveContributor`), inspects the request's host header (e.g., `tenant1.mydomain.com`).
3.  If the host matches the configured subdomain pattern (derived from `WebSiteRootAddress` with the `{TENANCY_NAME}` placeholder), the middleware extracts the tenancy name (`tenant1` in this example).
4.  It then attempts to find this tenant in the database.
5.  If found, the current tenant context is set for the duration of that request. This ensures data isolation and that tenant specific settings, services, and views (if customized per tenant) are used.

### MVC Application UI & URL Generation

1.  The MVC application is accessed via a URL like `https://tenant1.mydomain.com/SomePage`.
2.  The backend resolves the tenant (`tenant1`) as described above.
3.  When server side code (Controllers, Views, Tag Helpers, Razor Pages) generates URLs (e.g., for links, form actions, redirects), ASP.NET Core's routing and URL generation mechanisms are aware of the current request's scheme (`https`), host (`tenant1.mydomain.com`), and path base.
4.  Therefore, URLs generated by helpers like `@Url.Action("Index", "Home")` or `<a asp-action="Index" asp-controller="Home">Link</a>` will correctly form fully qualified or path relative URLs that maintain the tenant's subdomain (e.g., `/Home/Index` on `tenant1.mydomain.com` or `https://tenant1.mydomain.com/Home/Index`).
5.  This ensures that navigation within the tenant's site stays on the correct subdomain.

## 4. Troubleshooting Common Subdomain Multi Issues

Here are some common problems and how to address them:

### DNS Not Resolving

* **Propagation Time:** DNS changes (especially for new wildcard records) can take time to propagate globally (minutes to hours).
* **Record Verification:** Double check your DNS provider's settings. Ensure the `A` record or `CNAME` record for the wildcard (e.g., `*.mydomain.com`) correctly points to your web server's public IP address.
* **Local `hosts` File:** For local development, ensure your `hosts` file entries are correctly spelled, saved, and that your browser or OS is not caching old DNS lookups aggressively (try clearing cache or an incognito window).

### CORS Errors (Contextualized for MVC)

* While less common for a self contained MVC application serving its own views and assets, CORS errors can arise if:
    * Your MVC application makes client side (JavaScript) AJAX requests to a different domain (e.g., a separate `YourProjectName.Web.Host` API for specific tasks, though less typical if MVC is the primary UI).
    * You use a CDN for static assets that requires specific CORS headers.
* If such errors occur, check the `App:CorsOrigins` setting in the `appsettings.json` of the project *receiving* the request (e.g., `Web.Host` if it's the API being called).
* Inspect the browser's developer console (Network tab) for detailed CORS error messages.

### SSL Certificate Errors

* **Certificate Name Mismatch:** This means the SSL certificate presented by the server is not valid for the domain requested by the browser.
    * Ensure your certificate is a true wildcard (e.g., `*.mydomain.com`) or a SAN certificate that explicitly lists all required tenant subdomains or the relevant wildcard.
* **Untrusted Certificate:**
    * For production, ensure your certificate is issued by a reputable CA.
    * For local development with self signed or `mkcert` generated certificates, ensure the root CA certificate used to sign them is trusted by your operating system and browser.
* **Mixed Content:** Ensure all resources (CSS, JS, images, etc.) are loaded over HTTPS if your site is served via HTTPS.

### Incorrect Tenant Loaded or Redirected to Host

* **`WebSiteRootAddress` Configuration Check:**
    * Verify the `App:WebSiteRootAddress` in `YourProjectName.Web.Mvc/appsettings.json`.
    * Ensure it correctly uses the `{TENANCY_NAME}` placeholder and reflects your actual domain structure.
* **`PreventNotExistingTenantSubdomains`:**
    * Check this setting in `YourProjectConsts.cs` (in the `Core.Shared` project).
    * If `true` (which is the default and recommended for subdomain tenancy), requests to subdomains for tenants that don't exist will typically result in a "not found" or a redirect to a host/error page, rather than defaulting to the host tenant's content under the unrecognized subdomain.
* **Tenant Existence:** Ensure the tenancy name being extracted from the URL (e.g., `tenant1` from `tenant1.mydomain.com`) actually exists in your application's database and is active.
* **ASP.NET Core Logging:** Increase logging verbosity for ASP.NET Core, especially for categories like `Microsoft.AspNetCore.Hosting` and ASP.NET Zero's tenancy services, to see how the tenant is being identified (or failing to be identified) from the host header.