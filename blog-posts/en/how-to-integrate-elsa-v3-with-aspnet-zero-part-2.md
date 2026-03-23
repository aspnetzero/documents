**Title:** How to Integrate Elsa v3 with ASP.NET Zero - Part 2: Angular Frontend Integration
**Description:** This guide walks through the Angular frontend integration of Elsa v3 within an ASP.NET Zero solution. It covers connecting to the Elsa API, configuring authentication and API endpoints, integrating workflow UI components, and enabling seamless interaction with the backend workflow engine.

# How to Integrate Elsa v3 with ASP.NET Zero - Part 2: Angular Frontend Integration

In [Part 1]((https://aspnetzero.com/how-to-integrate-elsa-v3-with-aspnet-zero-part-1), we built the complete backend infrastructure: a standalone ElsaServer project with the Elsa v3 workflow engine, JWT authentication, multi-tenancy support, Blazor Server Elsa Studio, API authorization middleware, and auto-start hosted services. By the end of Part 1, we had a working ElsaServer that launches automatically with the ASP.NET Zero Web.Host and exposes both the Elsa API and the Blazor-based workflow designer.

In this second part, we'll complete the integration by embedding the Elsa Studio workflow designer directly into the ASP.NET Zero **Angular** application. Users will be able to access the designer from the sidebar menu without ever leaving the Angular app, the Blazor Studio loads inside an iframe with automatic JWT token passing.

## What We'll Build

By the end of this article, you'll have:

- A configuration entry for the ElsaServer URL in the Angular app
- A standalone Angular component that embeds the Elsa Studio via iframe
- Automatic JWT token forwarding from Angular to the Blazor Studio
- Route configuration with permission-based guards
- A sidebar navigation menu item under the main menu

Let's get started.

## Step 1: Add the ElsaServer URL to AppConsts

The Angular app needs to know where the ElsaServer is running so it can construct the iframe URL. In ASP.NET Zero's Angular project, application-wide constants are stored in `src/shared/AppConsts.ts`.

Add the `elsaServerUrl` static property:

```typescript
export class AppConsts {
    static readonly tenancyNamePlaceHolderInUrl = '{TENANCY_NAME}';

    static remoteServiceBaseUrl: string;
    static remoteServiceBaseUrlFormat: string;
    static appBaseUrl: string;
    static appBaseHref: string;
    static appBaseUrlFormat: string;
    static recaptchaSiteKey: string;
    static subscriptionExpireNootifyDayCount: number;
    
    // Elsa Workflow Server URL
    static elsaServerUrl: string = 'https://localhost:44311';

    // ... rest of the existing code
}
```

This URL should match the `Urls` value configured in the `ElsaServer` section of `Web.Host/appsettings.json` from Part 1. During development, this defaults to `https://localhost:44311`. For production deployments, you would update this to match your server's actual URL.

> **Tip:** The component we'll build in Step 2 includes a fallback mechanism, if `elsaServerUrl` is not set, it will derive the URL from `remoteServiceBaseUrl` by replacing the port with `44311`. But it's best to set it explicitly.

## Step 2: Create the Elsa Workflows Component

This is the core of the Angular integration, a standalone component that embeds the Elsa Studio Blazor Server application inside an iframe, automatically passing the current user's JWT token.

Create the folder `src/app/elsa/` and add three files.

### Component TypeScript - elsa-workflows.component.ts

```typescript
import { Component, OnInit, OnDestroy, ElementRef, AfterViewInit, signal, viewChild } from '@angular/core';
import { DomSanitizer, SafeResourceUrl } from '@angular/platform-browser';
import { AppConsts } from '@shared/AppConsts';
import { appModuleAnimation } from '@shared/animations/routerTransition';
import { SubHeaderComponent } from '@app/shared/common/sub-header/sub-header.component';
import { LocalizePipe } from '@shared/common/pipes/localize.pipe';

@Component({
    selector: 'app-elsa-workflows',
    templateUrl: './elsa-workflows.component.html',
    styleUrls: ['./elsa-workflows.component.less'],
    animations: [appModuleAnimation()],
    imports: [SubHeaderComponent, LocalizePipe],
})
export class ElsaWorkflowsComponent implements OnInit, OnDestroy, AfterViewInit {
    elsaFrame = viewChild<ElementRef<HTMLIFrameElement>>('elsaFrame');

    elsaStudioUrl = signal<SafeResourceUrl | null>(null);
    isLoading = signal(true);
    errorMessage = signal('');

    private messageHandler: (event: MessageEvent) => void;

    constructor(private sanitizer: DomSanitizer) {}

    ngOnInit(): void {
        this.initializeElsaStudio();
        this.setupMessageListener();
    }

    ngAfterViewInit(): void {
        // Frame loaded callback
    }

    ngOnDestroy(): void {
        if (this.messageHandler) {
            window.removeEventListener('message', this.messageHandler);
        }
    }

    private initializeElsaStudio(): void {
        const accessToken = abp.auth.getToken();

        if (!accessToken) {
            this.errorMessage.set('Authentication token not found. Please log in again.');
            this.isLoading.set(false);
            return;
        }

        const elsaServerUrl = this.getElsaServerUrl();
        const urlWithToken = `${elsaServerUrl}?token=${encodeURIComponent(accessToken)}`;

        this.elsaStudioUrl.set(this.sanitizer.bypassSecurityTrustResourceUrl(urlWithToken));
    }

    private getElsaServerUrl(): string {
        const configuredUrl = (AppConsts as any).elsaServerUrl;
        if (configuredUrl) {
            return configuredUrl;
        }

        if (AppConsts.remoteServiceBaseUrl) {
            const baseUrl = new URL(AppConsts.remoteServiceBaseUrl);
            return `${baseUrl.protocol}//${baseUrl.hostname}:44311`;
        }

        return 'https://localhost:44311';
    }

    private setupMessageListener(): void {
        this.messageHandler = (event: MessageEvent) => {
            const allowedOrigins = [this.getElsaServerUrl(), 'https://localhost:44311', 'http://localhost:44311'];

            if (
                !allowedOrigins.some((origin) => event.origin.startsWith(origin.split('://')[1]?.split(':')[0] || ''))
            ) {
                return;
            }

            if (event.data?.type === 'ELSA_STUDIO_READY') {
                this.onElsaStudioReady();
            } else if (event.data?.type === 'ELSA_STUDIO_ERROR') {
                this.onElsaStudioError(event.data.message);
            } else if (event.data?.type === 'ELSA_TOKEN_EXPIRED') {
                this.onTokenExpired();
            }
        };

        window.addEventListener('message', this.messageHandler);
    }

    private onElsaStudioReady(): void {
        this.isLoading.set(false);
    }

    private onElsaStudioError(message: string): void {
        this.isLoading.set(false);
        this.errorMessage.set(message || 'An error occurred while loading Elsa Studio');
    }

    private onTokenExpired(): void {
        this.errorMessage.set('Your session has expired. Please refresh the page or log in again.');
    }

    onFrameLoad(): void {
        if (this.isLoading()) {
            this.isLoading.set(false);
        }
    }

    onFrameError(): void {
        this.isLoading.set(false);
        this.errorMessage.set('Failed to load Elsa Studio. Please check if the Elsa Server is running.');
    }

    refreshToken(): void {
        const accessToken = abp.auth.getToken();
        if (accessToken && this.elsaFrame()?.nativeElement) {
            const elsaServerUrl = this.getElsaServerUrl();
            const urlWithToken = `${elsaServerUrl}?token=${encodeURIComponent(accessToken)}`;
            this.elsaFrame()!.nativeElement.src = urlWithToken;
            this.isLoading.set(true);
            this.errorMessage.set('');
        }
    }

    openInNewTab(): void {
        const accessToken = abp.auth.getToken();
        if (accessToken) {
            const elsaServerUrl = this.getElsaServerUrl();
            const urlWithToken = `${elsaServerUrl}?token=${encodeURIComponent(accessToken)}`;
            const newWindow = window.open(urlWithToken, '_blank', 'noopener,noreferrer');
            if (newWindow) {
                newWindow.opener = null;
            }
        }
    }
}

```

Let's walk through the key parts of this component:

**Token retrieval:** The component uses `abp.auth.getToken()`, this is ABP Framework's built-in method that retrieves the current user's JWT token from the cookie/storage. This is the same token that Web.Host issued during login.

**URL construction:** The ElsaServer URL is read from `AppConsts.elsaServerUrl`. The JWT token is appended as a `?token=` query parameter. As we covered in Part 1 (Step 5), the `_Host.cshtml` page on the ElsaServer extracts this token from the URL, stores it in localStorage, and passes it to SignalR's `accessTokenFactory` so the Blazor circuit can use it for API calls.

**Security sanitization:** Since Angular blocks dynamic URLs in iframes by default (to prevent XSS), we use `DomSanitizer.bypassSecurityTrustResourceUrl()` to explicitly mark the URL as safe. This is appropriate here because we're constructing the URL ourselves from trusted configuration values.

**PostMessage listener:** The component listens for `window.postMessage` events from the iframe. This allows the Blazor Studio to communicate status back to the Angular host, for example, signaling when it's ready, reporting errors, or notifying about expired tokens.

**Refresh and open-in-new-tab:** Two utility methods let users refresh the iframe (which re-sends a fresh token) or open the Elsa Studio in a separate browser tab.

### Component Template - elsa-workflows.component.html

```html
<div class="elsa-workflows-container">
    <sub-header [title]="'ElsaWorkflows' | localize" [description]="'ElsaWorkflowsDescription' | localize">
        <div role="actions">
            <button class="btn btn-sm btn-light-primary me-2" (click)="refreshToken()" [disabled]="isLoading()">
                <i class="fas fa-sync-alt me-1"></i>
                {{ 'Refresh' | localize }}
            </button>
            <button class="btn btn-sm btn-light-info" (click)="openInNewTab()">
                <i class="fas fa-external-link-alt me-1"></i>
                {{ 'OpenInNewTab' | localize }}
            </button>
        </div>
    </sub-header>

    <div class="card card-flush">
        <div class="card-body p-0">
            <!-- Loading Indicator -->
            @if (isLoading()) {
                <div class="elsa-loading-overlay">
                    <div class="loading-content">
                        <div class="spinner-border text-primary" role="status">
                            <span class="visually-hidden">Loading...</span>
                        </div>
                        <p class="mt-3 text-muted">
                            {{ 'LoadingElsaStudio' | localize }}
                        </p>
                    </div>
                </div>
            }
            <!-- Error Message -->
            @if (errorMessage()) {
                <div class="elsa-error-container">
                    <div class="alert alert-danger d-flex align-items-center" role="alert">
                        <i class="fas fa-exclamation-triangle me-3 fs-2"></i>
                        <div>
                            <h5 class="alert-heading mb-1">
                                {{ 'Error' | localize }}
                            </h5>
                            <p class="mb-0">{{ errorMessage() }}</p>
                        </div>
                    </div>
                    <div class="mt-4 text-center">
                        <button class="btn btn-primary" (click)="refreshToken()">
                            <i class="fas fa-redo me-2"></i>
                            {{ 'TryAgain' | localize }}
                        </button>
                    </div>
                </div>
            }
            <!-- Elsa Studio iFrame -->
            @if (elsaStudioUrl() && !errorMessage()) {
                <iframe
                    #elsaFrame
                    [src]="elsaStudioUrl()!"
                    class="elsa-studio-frame"
                    [class.loading]="isLoading()"
                    (load)="onFrameLoad()"
                    (error)="onFrameError()"
                    frameborder="0"
                    allowfullscreen></iframe>
            }
        </div>
    </div>
</div>
```

The template has three states:

1. **Loading**: Shows a spinner overlay while the Blazor Studio is initializing. The iframe is still visible underneath to give users a sense of progress.
2. **Error**: Displays an alert with the error message and a "Try Again" button. The iframe is hidden when an error occurs.
3. **Loaded**: The iframe fills the card body, showing the full Elsa Studio workflow designer.

The `<sub-header>` component is ASP.NET Zero's standard page header. It displays the page title and description, with a toolbar area on the right for the Refresh and Open in New Tab buttons.

> **Note:** The template uses Angular's new `@if` control flow syntax (Angular 17+). If your ASP.NET Zero version uses an older Angular version, replace `@if (condition) { ... }` with `<div *ngIf="condition">...</div>`.

### Component Styles - elsa-workflows.component.less

```less
.elsa-workflows-container {
    height: 100%;
    display: flex;
    flex-direction: column;
}

.elsa-studio-frame {
    width: 100%;
    height: calc(100vh - 200px);
    min-height: 600px;
    border: none;
    transition: opacity 0.3s ease;

    &.loading {
        opacity: 0.3;
    }
}

.elsa-loading-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(255, 255, 255, 0.9);
    z-index: 10;
    min-height: 400px;

    .loading-content {
        text-align: center;
    }

    .spinner-border {
        width: 3rem;
        height: 3rem;
    }
}

.elsa-error-container {
    padding: 40px;
    text-align: center;
    min-height: 400px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;

    .alert {
        max-width: 600px;
        text-align: left;
    }
}

.toolbar-actions {
    display: flex;
    align-items: center;
    gap: 8px;
}

// Card styling
.card {
    position: relative;
    flex: 1;
    
    .card-body {
        position: relative;
        height: 100%;
    }
}

// Responsive adjustments
@media (max-width: 768px) {
    .elsa-studio-frame {
        height: calc(100vh - 250px);
        min-height: 400px;
    }

    .toolbar-actions {
        flex-direction: column;
        gap: 4px;

        .btn {
            width: 100%;
        }
    }
}

// Dark mode support
:host-context(.dark-mode) {
    .elsa-loading-overlay {
        background: rgba(30, 30, 30, 0.9);
    }

    .elsa-error-container {
        .alert-danger {
            background: rgba(220, 53, 69, 0.1);
            border-color: rgba(220, 53, 69, 0.3);
        }
    }
}
```

Key styling details:

- The iframe uses `calc(100vh - 200px)` to fill the available viewport height, minus the header area. This ensures the workflow designer has maximum screen real estate.
- During loading, the iframe fades to 30% opacity with a smooth transition, while the spinner overlay appears on top.
- Responsive styles adjust the layout for mobile devices, the toolbar buttons stack vertically and the iframe gets a smaller minimum height.
- Dark mode support is included via `:host-context(.dark-mode)`, adjusting the overlay and error container backgrounds.

## Step 3: Configure Routing

Now we need to add a route so users can navigate to the Elsa Workflows page. In ASP.NET Zero's Angular app, the main application routes are defined in `src/app/app-routes.ts`.

Add an `elsa` route group as a child of the `app` route, alongside the existing `main` and `admin` routes:

```typescript
import { Routes } from '@angular/router';
import { AppRouteGuard } from './shared/common/auth/auth-route-guard';

export const appRoutes: Routes = [
    {
        path: 'app',
        loadComponent: () => import('./app.component').then((m) => m.AppComponent),
        canActivate: [AppRouteGuard],
        canActivateChild: [AppRouteGuard],
        children: [
            {
                path: '',
                children: [
                    {
                        path: 'notifications',
                        loadComponent: () =>
                            import('./shared/layout/notifications/notifications.component')
                                .then((m) => m.NotificationsComponent),
                    },
                    { path: '', redirectTo: '/app/main/dashboard', pathMatch: 'full' },
                ],
            },
            {
                path: 'main',
                data: { preload: true },
                children: [
                    {
                        path: 'dashboard',
                        loadComponent: () =>
                            import('./main/dashboard/dashboard.component').then((m) => m.DashboardComponent),
                        data: { permission: 'Pages.Tenant.Dashboard' },
                    },
                    { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
                ],
            },
            {
                path: 'admin',
                data: { preload: true },
                canLoad: [AppRouteGuard],
                children: [
                    {
                        path: 'users',
                        loadComponent: () => import('./admin/users/users.component').then((m) => m.UsersComponent),
                        data: { permission: 'Pages.Administration.Users' },
                    },

                    //...
                ]
            },
            // ===== Elsa Workflows Route =====
            {
                path: 'elsa',
                children: [
                    {
                        path: 'workflows',
                        loadComponent: () =>
                            import('@app/elsa/elsa-workflows.component')
                                .then((m) => m.ElsaWorkflowsComponent),
                        data: { permission: 'Pages.Elsa' },
                    },
                    {
                        path: '',
                        redirectTo: 'workflows',
                        pathMatch: 'full',
                    },
                ],
            },
            // =================================
            {
                path: '**',
                redirectTo: 'notifications',
            },
        ],
    },
];
```

Key points about this route configuration:

- **Lazy loading:** The component is loaded via `loadComponent` with a dynamic `import()`, so it's only downloaded when the user actually navigates to the Elsa Workflows page. This keeps the initial bundle small.
- **Permission guard:** The `data: { permission: 'Pages.Elsa' }` property works with ASP.NET Zero's `AppRouteGuard`. The guard checks whether the current user has the `Pages.Elsa` permission before allowing access. If the user doesn't have the permission, they're redirected away.
- **Route structure:** The route is at `/app/elsa/workflows`, placed as a sibling to `main` and `admin`, not nested under `admin`. This is a design choice; you could place it under `admin` if you prefer.
- **Default redirect:** Navigating to `/app/elsa` automatically redirects to `/app/elsa/workflows`.

## Step 4: Add the Navigation Menu Item

The final piece is adding an "Elsa Workflows" entry to the sidebar navigation menu. In ASP.NET Zero's Angular app, the menu is defined in `src/app/shared/layout/nav/app-navigation.service.ts`.

Open this file and add a new `AppMenuItem` entry at the end of the menu items array in the `getMenu()` method:

```typescript
getMenu(): AppMenu {
    return new AppMenu('MainMenu', 'MainMenu', [
        // ... existing menu items

        new AppMenuItem(
            'DemoUiComponents',
            'Pages.DemoUiComponents',
            'flaticon-shapes',
            '/app/admin/demo-ui-components'
        ),
        // ===== Elsa Workflows Menu Item =====
        new AppMenuItem(
            'ElsaWorkflows',
            'Pages.Elsa',
            'flaticon-network',
            '/app/elsa/workflows',
            [],
            [],
            false,
            {},
            undefined,
            true
        ),
    ]);
}
```

Let's break down each parameter of the `AppMenuItem` constructor:

| Parameter | Value | Description |
|---|---|---|
| `name` | `'ElsaWorkflows'` | The localization key used to display the menu label. ASP.NET Zero will look up this key in the localization source to get the translated text. |
| `permissionName` | `'Pages.Elsa'` | The permission required to see this menu item. If the user doesn't have this permission, the item is hidden. This matches the permission we defined in `AppPermissions.cs` in Part 1. |
| `icon` | `'flaticon-network'` | The CSS class for the menu icon. `flaticon-network` shows a network/workflow-like icon, which fits well for workflows. |
| `route` | `'/app/elsa/workflows'` | The Angular route this menu item navigates to. |
| `routeTemplates` | `[]` | Additional route templates that should highlight this menu item as active. Empty in our case. |
| `items` | `[]` | Child menu items. Empty since we don't have sub-menus. |
| `external` | `false` | Whether this links to an external URL. |
| `parameters` | `{}` | Additional route parameters. |
| `featureDependency` | `undefined` | A function that checks if a feature is enabled. Not used here. |
| `requiresAuthentication` | `true` | Whether the user must be logged in to see this item. Redundant when `permissionName` is set (since permissions imply authentication), but explicit is good. |

### How the Permission Visibility Works

ASP.NET Zero's `showMenuItem` method in `AppNavigationService` automatically handles menu visibility based on permissions. When the Angular app loads, it receives the list of granted permissions from the server. The `showMenuItem` method checks:

1. Is the user authenticated? (if `requiresAuthentication` is `true`)
2. Does the user have the required permission? (checks `permissionName` against `PermissionCheckerService`)
3. Is the feature dependency satisfied? (if applicable)

If any check fails, the menu item is hidden. This means only users with the `Pages.Elsa` permission will see the "Elsa Workflows" menu item in their sidebar.

### Adding Localization Strings

For the menu label and page header to display properly, you need to add localization entries. In ASP.NET Zero, localization XML files are located in the Core project under `Localization/YourProjectName/`.

Add these entries to the default language XML file:

```xml
<text name="ElsaWorkflows">Elsa Workflows</text>
<text name="ElsaWorkflowsDescription">Design and manage workflows using the Elsa Studio visual designer</text>
<text name="LoadingElsaStudio">Loading Elsa Studio...</text>
<text name="OpenInNewTab">Open in New Tab</text>
```

> **Note:** If you skip adding localization strings, the menu item and page header will display the raw key names (e.g., "ElsaWorkflows" instead of "Elsa Workflows"), which is still functional but less polished.

## Testing the Complete Integration

With all the pieces in place, let's verify everything works end-to-end.

### 1. Start the Backend

Make sure the ASP.NET Core backend is running. Web.Host will automatically start ElsaServer as a child process:

```bash
cd aspnet-core
dotnet run --project src/YourProjectName.Web.Host
```

You should see log output indicating both Web.Host and ElsaServer have started:

```
Web.Host listening on https://localhost:44301
[ElsaServer] Process started (PID: XXXXX)
[ElsaServer] Listening on https://localhost:44311
```

### 2. Start the Angular App

In a separate terminal:

```bash
cd angular
npm start
```

The Angular app will start on `http://localhost:4200`.

### 3. Verify the Workflow

1. Open `http://localhost:4200` in your browser
2. Log in with an admin account (which has the `Pages.Elsa` permission)
3. Look for the **"Elsa Workflows"** menu item in the sidebar (with the network icon)
4. Click it, you should see the Elsa Studio loading inside an iframe
5. After a few seconds, the full workflow designer should appear
6. Try creating a new workflow to verify the Elsa API is accessible

![Elsa v3 Workflows Page](/Images/Blog/elsa-v3-workflows-page.png)

### Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Menu item not visible | User lacks `Pages.Elsa` permission | Grant the permission via Roles in the admin panel |
| Iframe shows blank page | ElsaServer not running | Check if Web.Host started the child process; verify `https://localhost:44311` is accessible |
| "Authentication token not found" error | User not logged in or token expired | Log out and log back in |
| CORS errors in browser console | ElsaServer origin not in CORS list | Ensure `https://localhost:44311` is in `App:CorsOrigins` in Web.Host's `appsettings.json` |
| Blazor connection error | SignalR can't connect with token | Check that the JWT `SecurityKey`, `Issuer`, and `Audience` match between Web.Host and ElsaServer |

## Summary

In this two-part series, we built a complete integration between the **Elsa v3 workflow engine** and **ASP.NET Zero**.

This architecture provides a clean separation of concerns while maintaining a seamless user experience. The ElsaServer runs as an independent process, which means it can be scaled, deployed, and updated independently from the main ASP.NET Zero application. Yet from the user's perspective, the workflow designer feels like a native part of the Angular application.

We hope this guide helps you integrate powerful workflow capabilities into your ASP.NET Zero projects. Happy workflow building!

