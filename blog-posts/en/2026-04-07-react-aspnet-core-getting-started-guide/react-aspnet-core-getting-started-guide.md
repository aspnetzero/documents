# React + ASP.NET Core: Enterprise Project Getting Started Guide

If you are building a modern enterprise web application in 2026, **React with ASP.NET Core** is one of the most practical stacks you can choose. React gives you a flexible component model, excellent ecosystem support, and strong performance with a Vite-based workflow. ASP.NET Core gives you a high-performance backend, robust security primitives, and a mature architecture for APIs.

In this guide, we will walk through **three different ways** to start a React + ASP.NET Core project:

1. Manual setup with Vite + ASP.NET Core Web API
2. Visual Studio template setup
3. ASP.NET Zero React UI setup for enterprise-grade applications

By the end, you will know which approach is best for your team and timeline.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Approach 1 - Manual Setup](#approach-1---manual-setup)
  - [Step 1: Create the React App with Vite](#step-1-create-the-react-app-with-vite)
  - [Step 2: Create the ASP.NET Core Web API](#step-2-create-the-aspnet-core-web-api)
  - [Step 3: Configure CORS](#step-3-configure-cors)
  - [Step 4: Configure the Vite Proxy](#step-4-configure-the-vite-proxy)
  - [Step 5: Make the First API Call from React](#step-5-make-the-first-api-call-from-react)
- [Approach 2 - Visual Studio Template](#approach-2---visual-studio-template)
- [Approach 3 - ASP.NET Zero React UI](#approach-3---aspnet-zero-react-ui)
- [Comparison](#comparison)
- [Common Issues and Solutions](#common-issues-and-solutions)
- [Next Steps](#next-steps)

## Prerequisites

Before we start, install and verify the following tools:

- **Node.js** 24.x or later - [download here](https://nodejs.org/)
- **Vite** (installed per project, no global install required)
- **.NET 10 SDK** - [download here](https://dotnet.microsoft.com/download/dotnet/10.0)
- **Visual Studio 2026** or **VS Code**
- Optional but recommended: **pnpm** or **yarn** for faster dependency management

Run these commands to verify your environment:

```bash
node --version
npm --version
dotnet --version
```

You should see Node.js 24+, npm 11+, and .NET 10.x.

## Approach 1 - Manual Setup

This approach gives you full control and helps you understand each moving part. It is ideal when you want to shape your own architecture from day one.

### Step 1: Create the React App with Vite

Open a terminal and run:

```bash
npm create vite@latest myapp.client -- --template react-ts
cd myapp.client
npm install
npm run dev
```

Open `http://localhost:5173`. You should see the Vite + React starter page.

We use the `react-ts` template because TypeScript is a better default for enterprise codebases. It improves refactoring safety and makes large teams more productive.

### Step 2: Create the ASP.NET Core Web API

Open a second terminal in the parent folder:

```bash
dotnet new webapi -n MyApp.Api --use-controllers
cd MyApp.Api
dotnet run
```

The API starts on a port shown in terminal output, usually like `http://localhost:52xx` and `https://localhost:72xx`.

The default `WeatherForecastController` is enough for the first integration test. Open this URL in your browser to verify the API:

- `http://localhost:{port}/WeatherForecast`

If you can see JSON data, backend setup is ready.

### Step 3: Configure CORS

React and ASP.NET Core run on different origins in development. The browser blocks cross-origin calls unless we explicitly allow them.

Open `Program.cs` in the API project and update it like this:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddOpenApi();

builder.Services.AddCors(options =>
{
    options.AddPolicy("ReactDev", policy =>
    {
        policy.WithOrigins("http://localhost:5173")
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
app.UseCors("ReactDev");
app.UseAuthorization();
app.MapControllers();
app.Run();
```

**Note:** Keep this policy strict in production. Only allow your real frontend domain.

### Step 4: Configure the Vite Proxy

CORS works, but during development a proxy is cleaner. Your frontend calls `/api/*`, and Vite forwards those requests to ASP.NET Core.

Open `vite.config.ts` in your React project and add proxy settings:

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/WeatherForecast': {
        target: 'http://localhost:5266',
        changeOrigin: true,
        secure: false,
      },
    },
  },
})
```

Replace `5266` with your actual API HTTP port.

Restart Vite after this change:

```bash
npm run dev
```

### Step 5: Make the First API Call from React

Now let us fetch data from ASP.NET Core and render it in React.

Update `src/App.tsx`:

```tsx
import { useEffect, useState } from 'react'
import './App.css'

type WeatherForecast = {
  date: string
  temperatureC: number
  temperatureF: number
  summary: string
}

function App() {
  const [forecasts, setForecasts] = useState<WeatherForecast[]>([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    fetch('/WeatherForecast')
      .then(async response => {
        if (!response.ok) {
          throw new Error(`Request failed: ${response.status}`)
        }
        const data = (await response.json()) as WeatherForecast[]
        setForecasts(data)
      })
      .catch(err => setError(err.message))
      .finally(() => setLoading(false))
  }, [])

  if (loading) {
    return <p>Loading weather data...</p>
  }

  if (error) {
    return <p>Error: {error}</p>
  }

  return (
    <main style={{ padding: 24 }}>
      <h1>Weather Forecast</h1>
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
          {forecasts.map(item => (
            <tr key={item.date}>
              <td>{item.date}</td>
              <td>{item.temperatureC}</td>
              <td>{item.temperatureF}</td>
              <td>{item.summary}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </main>
  )
}

export default App
```

Save the file and refresh the page. If everything is configured correctly, your React app will display forecast data from ASP.NET Core.

At this point, your baseline architecture is ready.

## Approach 2 - Visual Studio Template

If you prefer a faster start with less manual setup, use the built-in template from Visual Studio.

1. Open Visual Studio 2026.
2. Select **Create a new project**.
3. Search for **ASP.NET Core with React**.
4. Choose **.NET 10** as target framework.
5. Create and run the project.

The template gives you:

- A ready ASP.NET Core backend
- A React frontend project integrated into the solution
- Launch profiles for local debugging
- Basic dev-time wiring between frontend and backend

This is a good option for PoCs and small-to-medium internal apps.

What is still missing for most enterprise projects:

- End-to-end authentication and authorization flows
- Multi-tenancy
- Role/permission management UI
- Audit logging and operational tooling
- Accelerated CRUD scaffolding

You can build all of these, but it takes significant time.

## Approach 3 - ASP.NET Zero React UI

If your goal is to ship an enterprise product quickly, this is the highest-leverage option.

[ASP.NET Zero](https://aspnetzero.com/) provides a production-ready architecture with **official React UI support** (available since v15.1) and **RAD capabilities in v15.2**.

### Quick Start

1. Go to [aspnetzero.com](https://aspnetzero.com/) and get your package.
2. Select the **React** option while downloading.
3. Open the solution.
4. Update connection strings in `appsettings.json`.
5. Run the migrator project.
6. Start the backend host.
7. Start the React UI project.

Example run flow:

```bash
cd src/MyCompanyName.AbpZeroTemplate.Migrator
dotnet run

cd ../MyCompanyName.AbpZeroTemplate.Web.Host
dotnet run

cd ../../react
npm install
npm run dev
```

Then open your local React URL and sign in with seeded admin credentials.

For detailed environment-specific setup, follow the official docs:

- React getting started docs: [docs.aspnetzero.com](https://docs.aspnetzero.com/aspnet-core-react/latest/Getting-Started-React)

### Why This Matters for Enterprise Teams

With ASP.NET Zero React UI, you start with critical features already built:

- Authentication and authorization
- Role and permission management
- Tenant management and subscriptions
- Audit logs and notifications
- Production-ready UI infrastructure
- Battle-tested architecture conventions

### v15.2 RAD Advantage

The v15.2 RAD (Rapid Application Development) flow significantly reduces repetitive coding.

You define an entity once, and tooling generates:

- Backend entity and migration
- DTOs and application service methods
- API endpoints
- React list/create/edit pages
- Boilerplate integration code

This means your team spends time on business logic, not repeated scaffolding.

## Comparison

Here is a practical side-by-side comparison:

| Criteria | Manual Setup | VS Template | ASP.NET Zero React UI |
|---|---|---|---|
| Setup speed | Medium | Fast | Fast |
| Architecture control | Full | High | Full (source included) |
| Learning value | Very high | Medium | High |
| Built-in auth | No | No | Yes |
| Multi-tenancy | No | No | Yes |
| Role/permission UI | No | No | Yes |
| Production-ready admin UI | No | Basic | Yes (Metronic-based) |
| CRUD acceleration | No | No | Yes (RAD/Power Tools) |
| Best for | Custom frameworks, education | Prototypes | Enterprise delivery |

**Bottom line:**

- Choose **manual setup** when you want complete transparency and control.
- Choose **VS template** when you need a quick, clean starting point.
- Choose **ASP.NET Zero React UI** when delivery speed and enterprise features are top priority.

## Common Issues and Solutions

### Vite Proxy Returns 404

**Symptom:** Calls to `/WeatherForecast` return 404.

**Why it happens:** Proxy path does not match backend route or API port is wrong.

**Fix:**

- Verify backend route in your controller.
- Verify the target port in `vite.config.ts`.
- Restart Vite after config changes.

### HMR Not Refreshing Correctly

**Symptom:** UI changes are not reflected immediately.

**Why it happens:** Stale dev server state or package mismatch.

**Fix:**

```bash
rm -rf node_modules package-lock.json
npm install
npm run dev
```

If you are on Windows PowerShell, use:

```powershell
Remove-Item node_modules -Recurse -Force
Remove-Item package-lock.json -Force
npm install
npm run dev
```

### CORS Error in Browser Console

**Symptom:** `blocked by CORS policy`.

**Why it happens:** Missing or incorrect CORS policy in ASP.NET Core.

**Fix:** Ensure you registered and applied the policy in correct order:

```csharp
app.UseHttpsRedirection();
app.UseCors("ReactDev");
app.UseAuthorization();
```

### Authentication Integration Complexity

**Symptom:** Login appears to work, but protected API calls fail.

**Why it happens:** Missing token forwarding or inconsistent token storage strategy.

**Fix:**

- Centralize API calls with a single HTTP client layer.
- Add request interceptors to attach bearer tokens.
- Implement refresh token handling early.

If you want this fully prebuilt, ASP.NET Zero already includes this flow.

## Next Steps

Now your React + ASP.NET Core baseline is ready. Continue with one of these paths:

1. Add JWT authentication and protected routes.
2. Add CRUD modules with server-side validation and pagination.
3. Introduce role-based authorization and admin screens.
4. Evaluate ASP.NET Zero React UI if you want to skip infrastructure coding.

Useful links:

- ASP.NET Core docs: [learn.microsoft.com](https://learn.microsoft.com/en-us/aspnet/core/)
- ASP.NET Zero React docs: [docs.aspnetzero.com](https://docs.aspnetzero.com/aspnet-core-react/latest/Getting-Started-React)
- ASP.NET Zero React page: [aspnetzero.com/react](https://aspnetzero.com/react)

Building enterprise software is not only about choosing a frontend library. It is about reducing risk, increasing delivery speed, and keeping architecture maintainable as the product grows. React + ASP.NET Core is an excellent foundation, and with the right starting point, your team can move much faster.