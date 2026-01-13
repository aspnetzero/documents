# The State of .NET Auth in 2026

If you are a .NET developer building a modern web application in 2026, the question "How do I handle authentication?" is more complicated than it was five years ago.

For a long time, **IdentityServer** was the default answer. It was included in the project templates, it was free, and it worked. Then came the shift to **Duende IdentityServer**, a commercial pivot that—while sustainable for the maintainers—left many developers looking for alternatives, especially for smaller projects or startups with tight budgets.

Today, the landscape is fragmented but rich. You have native libraries, polished SaaS products, and powerful open-source servers.

In this post, we’ll explore the four main options for your .NET authentication stack in 2026: **OpenIddict**, **Keycloak**, **ASP.NET Core Identity (API Endpoints)**, and **Azure Entra ID**.

## 1. The Native Powerhouse: OpenIddict

If you want to build an OAuth2/OIDC server *inside* your ASP.NET Core application (just like you did with IdentityServer), **OpenIddict** is the de facto standard today.

It is a framework, not a turnkey solution. This means you have total control over your entities, your database schema (EF Core is fully supported), and your UI.

**Why choose it?**
- **Native .NET:** No separate containers to manage; it runs in your existing process.
- **Flexibility:** It supports almost every OAuth2/OIDC flow imaginable.
- **Cost:** It is open-source (Apache 2.0).

**How it looks in code:**
Wiring up OpenIddict is remarkably concise in modern .NET:

```csharp
// Program.cs
builder.Services.AddOpenIddict()
    .AddCore(options =>
    {
        options.UseEntityFrameworkCore()
               .UseDbContext<ApplicationDbContext>();
    })
    .AddServer(options =>
    {
        options.SetTokenEndpointUris("/connect/token");
        options.AllowClientCredentialsFlow();
        options.AddDevelopmentEncryptionCertificate()
               .AddDevelopmentSigningCertificate();
        options.UseAspNetCore()
               .EnableTokenEndpointPassthrough();
    })
    .AddValidation(options =>
    {
        options.UseLocalServer();
        options.UseAspNetCore();
    });
```

## 2. The "Simple" Choice: ASP.NET Core Identity (API Endpoints)

Not every app needs to be an OAuth2 Provider. If you are building a React/Angular/Blazor frontend that talks to a single .NET backend (a classic monolith structure), implementing full OIDC is often over-engineering.

Since .NET 8, Microsoft introduced **Identity API Endpoints**. These are pre-built endpoints (`/login`, `/register`, `/refresh`) that issue simple Bearer tokens or Cookies.

**Why choose it?**
- **Simplicity:** Zero external dependencies.
- **Speed:** You can set this up in 5 minutes.
- **Lightweight:** No complexity of scopes, grants, or discovery documents.

**How it looks in code:**
```csharp
// Program.cs
builder.Services.AddAuthorization();
builder.Services.AddIdentityApiEndpoints<IdentityUser>()
    .AddEntityFrameworkStores<ApplicationDbContext>();

var app = builder.Build();

// This adds /register, /login, /refresh automatically
app.MapIdentityApi<IdentityUser>(); 

app.Run();
```

## 3. The Container Standard: Keycloak

If your architecture involves multiple services (perhaps some in Node.js, Go, or Python) or you simply want to separate the "Identity concerns" from your "App logic," **Keycloak** is the industry standard open-source Identity Provider (IdP).

It is a Java application that you typically run in a Docker container.

**Why choose it?**
- **Features:** It has everything out of the box—UI for login, user management, 2FA, social login, etc.
- **Decoupling:** Your authentication server is a separate piece of infrastructure.
- **Ecosystem:** Massive community support.

**The Trade-off:**
You are now managing a Java service. You need to handle its database, updates, and scaling separately from your .NET app.

## 4. The Enterprise Standard: Azure Entra ID

Formerly known as **Azure Active Directory (Azure AD)**, Entra ID is the backbone of the Microsoft Cloud. If you are building B2B applications or internal corporate tools, this is often the path of least resistance.

For B2C (customer-facing) apps, **Microsoft Entra External ID** is the evolution of Azure AD B2C.

**Why choose it?**
- **Security:** Microsoft handles the security hardening.
- **Integration:** Seamless if you are hosting on Azure.
- **Compliance:** SOC2, HIPAA, GDPR—it’s all handled for you.

**The Trade-off:**
Configuration can be complex, and you are tied to the Azure ecosystem.

## Comparison Matrix

<table>
  <thead>
    <tr>
      <th>Feature</th>
      <th>OpenIddict</th>
      <th>Keycloak</th>
      <th>ASP.NET Identity</th>
      <th>Azure Entra ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Type</strong></td>
      <td>Library (Framework)</td>
      <td>Server (Container)</td>
      <td>Library (Built-in)</td>
      <td>SaaS</td>
    </tr>
    <tr>
      <td><strong>Cost</strong></td>
      <td>Free (Apache 2.0)</td>
      <td>Free (Apache 2.0)</td>
      <td>Free (MIT)</td>
      <td>Free Tier / Paid</td>
    </tr>
    <tr>
      <td><strong>Hosting</strong></td>
      <td>Self-Hosted (In-app)</td>
      <td>Self-Hosted (Docker)</td>
      <td>Self-Hosted (In-app)</td>
      <td>Cloud (Microsoft)</td>
    </tr>
    <tr>
      <td><strong>Complexity</strong></td>
      <td>High (High Control)</td>
      <td>Medium (Config)</td>
      <td>Low</td>
      <td>Medium/High</td>
    </tr>
    <tr>
      <td><strong>Best For</strong></td>
      <td>Custom Auth Needs, SaaS</td>
      <td>Microservices, Polyglot</td>
      <td>Simple Monoliths</td>
      <td>Enterprise, B2B</td>
    </tr>
  </tbody>
</table>


## Verdict: Which one should you choose?

1.  **Choose ASP.NET Core Identity** if you are building a straightforward app with a frontend and backend in the same domain and don't need to share tokens with third parties.
2.  **Choose OpenIddict** if you need a full OIDC server embedded in your .NET app, want to customize every aspect of the flow, or are building a SaaS that requires custom token logic.
3.  **Choose Keycloak** if you want a battle-tested, standalone server and are comfortable managing a Docker container.
4.  **Choose Azure Entra ID** if you are building internal enterprise apps or heavily invested in the Azure ecosystem and don't want to manage auth infrastructure.

The "one size fits all" era of IdentityServer is over. In 2026, the best choice depends entirely on your specific architecture.
