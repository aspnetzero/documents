# Angular + ASP.NET Core: Enterprise Project Getting Started Guide (2026)

If you are building an enterprise web application in 2026, the combination of **Angular** and **ASP.NET Core** is one of the strongest choices you can make. Angular gives you TypeScript safety, a powerful component architecture, and a mature ecosystem. ASP.NET Core gives you a high-performance, cross-platform backend with first-class dependency injection and a rich middleware pipeline.

In this post, we will walk through **three different ways** to start an Angular + ASP.NET Core project — from a fully manual setup to a one-click enterprise starter. By the end, you will have a clear understanding of which approach fits your project best.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Approach 1 — Manual Setup](#approach-1--manual-setup)
  - [Step 1: Create the Angular App](#step-1-create-the-angular-app)
  - [Step 2: Create the ASP.NET Core Web API](#step-2-create-the-aspnet-core-web-api)
  - [Step 3: Configure CORS](#step-3-configure-cors)
  - [Step 4: Set Up the Angular Proxy](#step-4-set-up-the-angular-proxy)
  - [Step 5: Make the First API Call](#step-5-make-the-first-api-call)
- [Approach 2 — Visual Studio Template](#approach-2--visual-studio-template)
- [Approach 3 — ASP.NET Zero](#approach-3--aspnet-zero)
- [Comparison](#comparison)
- [Common Issues and Solutions](#common-issues-and-solutions)
- [Next Steps](#next-steps)

## Prerequisites

Before we start, make sure you have the following tools installed:

- **Node.js** 24.x or later — [download here](https://nodejs.org/)
- **Angular CLI** 21.x — install with `npm install -g @angular/cli`
- **.NET 10 SDK** — [download here](https://dotnet.microsoft.com/download/dotnet/10.0)
- **Visual Studio 2026** (Latest stable version) or **VS Code** with the C# Dev Kit extension
- A terminal you are comfortable with (PowerShell, bash, or the VS Code integrated terminal)

You can verify your setup by running:

```bash
node --version    # v24.x (use latest stable)
ng version        # Angular CLI 21.x
dotnet --version  # 10.x
```

## Approach 1 — Manual Setup

This approach gives you full control over every detail. It is the best choice if you want to understand how the pieces fit together or if you have specific architectural requirements.

### Step 1: Create the Angular App

Open a terminal and run:

```bash
ng new MyApp.Client --style=scss --routing=true --ssr=false
cd MyApp.Client
ng serve
```

Navigate to `http://localhost:4200` — you should see the default Angular welcome page.

### Step 2: Create the ASP.NET Core Web API

Open a new terminal in the parent directory and create the API project:

```bash
dotnet new webapi -n MyApp.Api --use-controllers
cd MyApp.Api
```

Let's add a simple endpoint to test the connection. Open `Controllers/WeatherForecastController.cs` — the template already includes a `GET /WeatherForecast` endpoint that returns random forecast data. We will use this for our first integration test.

Run the API:

```bash
dotnet run
```

The API will start on a port shown in the terminal output (e.g. `http://localhost:5266`). You can verify it by navigating to `http://localhost:{port}/WeatherForecast` in your browser. Take note of this port — you will need it for the proxy configuration in Step 4.

### Step 3: Configure CORS

By default, your Angular app on `localhost:4200` cannot call the API on `localhost:5001` because of the browser's same-origin policy. We need to enable CORS on the API side.

Open `Program.cs` and add the following:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddOpenApi();

// Add CORS policy
builder.Services.AddCors(options =>
{
    options.AddPolicy("Angular", policy =>
    {
        policy.WithOrigins("http://localhost:4200")
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
}

app.UseHttpsRedirection();

// Use the CORS policy
app.UseCors("Angular");

app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Note:** This CORS configuration is for development only. In production, you should restrict the allowed origins to your actual domain.

### Step 4: Set Up the Angular Proxy

While CORS works, a cleaner approach during development is to use Angular's built-in proxy. This way, the Angular dev server forwards API requests to the backend, and you avoid CORS entirely.

Create a file called `proxy.conf.json` in the root of your Angular project:

```json
{
  "/WeatherForecast": {
    "target": "http://localhost:5266",
    "secure": false,
    "changeOrigin": true
  }
}
```

**Note:** Replace `5266` with the actual port your API is running on. You can find it in the terminal output when you run `dotnet run` — look for the line that says `Now listening on: http://localhost:XXXX`. Also, we are proxying the `/WeatherForecast` path directly because the default .NET template does not use an `/api` prefix. In a real project, you would typically add a common prefix like `/api` to all your controllers (using `[Route("api/[controller]")]`) and proxy that single prefix instead.

Update `angular.json` to use the proxy. Find the `serve` section and add the `proxyConfig` option:

```json
"serve": {
  "options": {
    "proxyConfig": "proxy.conf.json"
  }
}
```

Now restart the Angular dev server with `ng serve`. Any request to `/WeatherForecast` from your Angular app will be forwarded to the .NET backend.

### Step 5: Make the First API Call

Let's update the root component to fetch weather data from the API. First, we need to configure `HttpClient`. Open `src/app/app.config.ts` and add `provideHttpClient()`:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient()
  ]
};
```

Now open `src/app/app.ts` and update it:

```typescript
import { Component, inject, OnInit, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';

interface WeatherForecast {
  date: string;
  temperatureC: number;
  temperatureF: number;
  summary: string;
}

@Component({
  selector: 'app-root',
  templateUrl: './app.html',
  styleUrl: './app.scss'
})
export class App implements OnInit {
  private http = inject(HttpClient);
  forecasts = signal<WeatherForecast[]>([]);

  ngOnInit() {
    this.http.get<WeatherForecast[]>('/WeatherForecast')
      .subscribe(data => this.forecasts.set(data));
  }
}
```

Then update `src/app/app.html` with the template:

```html
<h1>Weather Forecast</h1>
@if (forecasts().length > 0) {
  <table>
    <thead>
      <tr>
        <th>Date</th>
        <th>Temp (C)</th>
        <th>Temp (F)</th>
        <th>Summary</th>
      </tr>
    </thead>
    <tbody>
      @for (f of forecasts(); track f.date) {
        <tr>
          <td>{{ f.date }}</td>
          <td>{{ f.temperatureC }}</td>
          <td>{{ f.temperatureF }}</td>
          <td>{{ f.summary }}</td>
        </tr>
      }
    </tbody>
  </table>
}
```

Save the file and check your browser. You should see the weather data rendered in a table. Your Angular frontend is now talking to your ASP.NET Core backend.

## Approach 2 — Visual Studio Template

If you prefer a faster start with less manual configuration, Visual Studio 2022 ships with a built-in **Angular + ASP.NET Core** project template.

1. Open Visual Studio 2026 and select **Create a new project**.
2. Search for **"ASP.NET Core with Angular"** and select the template.
3. Configure the project name, location, and solution name.
4. On the next screen, select **.NET 10** as the target framework.
5. Click **Create**.

Visual Studio generates a solution with two projects:

- An **ASP.NET Core** backend with a sample `WeatherForecast` API.
- An **Angular** frontend with proxy configuration already wired up.

Press **F5** to run the project. Both the backend and the frontend will start together, and you will see a working Angular app fetching data from the API.

**What you get out of the box:**
- Pre-configured proxy (`proxy.conf.js`)
- HTTPS development certificate setup
- Integrated launch profiles for debugging both projects simultaneously

**What you don't get:**
- Authentication / authorization
- Multi-tenancy
- Role management
- A production-ready UI framework

This template is a great starting point for prototypes or simple applications. For enterprise projects, you will need to add these features yourself — or use a framework that includes them. But, be aware that this option may have problems time to time because some pf the NPM packages intorduce breaking changes and it takes time for maintainer team to release a working version.

## Approach 3 — ASP.NET Zero

[ASP.NET Zero](https://aspnetzero.com/) is a production-ready starter kit built on ASP.NET Core that supports **Angular**, **React**, and **MVC (Razor)** as frontend options. It is designed specifically for enterprise applications and includes dozens of features out of the box.

Here is how to get started:

1. Go to [aspnetzero.com](https://aspnetzero.com/) and sign up for a license.
2. Navigate to the **Download** page, select **Angular** as your frontend, and download the project.
3. Open the solution in Visual Studio or VS Code.
4. Update the database connection string in `appsettings.json`.
5. Run the `migrator` project to create and seed the database:

```bash
cd src/MyProject.Migrator
dotnet run
```

6. Start the backend:

```bash
cd src/MyProject.Web.Host
dotnet run
```

7. Start the Angular frontend:

```bash
cd angular
yarn 
yarn create-dynamic-bundles
yarn start
```

Navigate to `http://localhost:4200` and log in with the default admin credentials. You will see a fully working enterprise application.

To run the project propertly, you can follow [Angular Getting Started Document](https://docs.aspnetzero.com/aspnet-core-angular/latest/Getting-Started-Angular).

**What you get out of the box:**

- **Authentication & Authorization:** Login, registration, two-factor authentication, social logins, LDAP/Active Directory integration — all pre-built and configurable.
- **Multi-Tenancy:** Host/tenant architecture with separate databases or shared database options. Tenant registration, subscription management, and edition management are built in.
- **Role & Permission Management:** A granular permission system with role-based and user-based authorization.
- **Organization Units:** Hierarchical organizational structure with the ability to assign users and roles per unit.
- **Audit Logging:** Every action is logged with user, timestamp, and change details.
- **Real-Time Notifications:** Built-in notification system with SignalR integration.
- **UI Theme:** A polished admin theme based on Metronic with responsive design, dark mode, and multiple layout options.
- **Power Tools:** A Visual Studio extension and a CLI tool that generates **entities, DTOs, application services, API controllers, Angular components, and unit tests** from a simple entity definition. Instead of writing CRUD code manually, you describe your entity and Power Tools generates everything.

Let's see Power Tools in action. Say you want to add a `Product` entity to your application. You define the entity fields (Name, Price, Category, etc.) in Power Tools, click **Generate**, and it creates:

- The `Product` entity and database migration
- `ProductAppService` with full CRUD operations
- `ProductDto` and `CreateProductDto` classes
- Angular components for listing, creating, and editing products
- Unit tests for the app service
- UI tests for your Angular pages

What would take hours of boilerplate coding is done in under a minute.

## Comparison

Here is how the three approaches compare:

| Feature | Manual Setup | VS Template | ASP.NET Zero |
|---|---|---|---|
| Setup time | 30-60 min | 5 min | 10 min |
| Authentication | You build it | You build it | Built-in |
| Authorization / Roles | You build it | You build it | Built-in |
| Multi-tenancy | You build it | You build it | Built-in |
| Organization units | You build it | You build it | Built-in |
| Audit logging | You build it | You build it | Built-in |
| UI theme | None | Basic | Metronic (production-ready) |
| Code generation tools | None | None | Power Tools |
| Real-time notifications | You build it | You build it | Built-in |
| Learning value | High | Medium | High |
| Flexibility | Full | Full | Full (source code included) |
| Cost | Free | Free | Licensed |

**Bottom line:** If you are learning or building a simple prototype, the manual setup or VS template is perfect. If you are building an enterprise application and want to skip months of infrastructure work, ASP.NET Zero gives you a massive head start.

## Common Issues and Solutions

### CORS Errors

**Symptom:** The browser console shows `Access to XMLHttpRequest has been blocked by CORS policy`.

**Solution:** Make sure your `Program.cs` includes the CORS middleware and that `app.UseCors()` is called **before** `app.UseAuthorization()`. Also verify that the origin URL matches exactly (including the port number and protocol).

```csharp
// Correct order
app.UseCors("Angular");
app.UseAuthorization();
```

### Proxy Not Working

**Symptom:** API calls return 404 or the Angular app tries to serve the request itself.

**Solution:** Double-check the `proxy.conf.json` path in `angular.json`. Make sure the `target` URL matches the port your API is running on. Restart `ng serve` after any proxy configuration change — the proxy config is only read at startup.

### SSL Certificate Errors

**Symptom:** `ERR_CERT_AUTHORITY_INVALID` or proxy shows `UNABLE_TO_VERIFY_LEAF_SIGNATURE`.

**Solution:** For development, set `"secure": false` in your `proxy.conf.json`. This tells the Angular dev server to accept self-signed certificates. Alternatively, trust the .NET development certificate:

```bash
dotnet dev-certs https --trust
```

### Angular CLI Version Mismatch

**Symptom:** Build errors or unexpected behavior after creating the project.

**Solution:** Make sure your global Angular CLI version matches the version in the project's `package.json`. You can check with `ng version` and update with:

```bash
npm install -g @angular/cli@19
```

## Next Steps

Now that your Angular + ASP.NET Core project is up and running, here are some next steps depending on which approach you chose:

- **Add authentication:** If you went with the manual setup or VS template, check out the [ASP.NET Core Identity documentation](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity) to add login and registration.
- **Explore ASP.NET Zero:** If you are interested in the enterprise approach, [contact](https://aspnetzero.com/contact) ASP.NET Zero Team and explore the [documentation](https://docs.aspnetzero.com/). The Angular getting started guide will have you running in under 10 minutes.
- **Try Power Tools:** If you are already using ASP.NET Zero, generate your first entity with [Power Tools](https://docs.aspnetzero.com/development-guide/rad-tool) and see how much time it saves.

Building enterprise applications is hard. Choosing the right foundation makes it significantly easier. Whether you start from scratch or stand on the shoulders of a mature framework, Angular and ASP.NET Core together give you a rock-solid stack for 2026 and beyond.
