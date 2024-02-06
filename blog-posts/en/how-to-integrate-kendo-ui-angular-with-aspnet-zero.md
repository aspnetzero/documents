# How to Integrate Kendo UI Angular with ASP.NET Zero

Learn how to integrate Kendo UI Angular, a comprehensive UI library, with ASP.NET Zero, a popular full stack .NET Core template based [ABP](https://aspnetboilerplate.com/). This integration enables the creation of modern, powerful web applications with enhanced productivity and performance.

## Create a New ASP.NET Zero Project

First, create a new project using the [ASP.NET Zero](https://aspnetzero.com/Download). 

* Select the `ASP.NET Core & Angular` template.
* You can choose single solution option if you want to have the backend and frontend in a single solution.

![kendo-ui-angular-tutorial](images/Blog/kendo-ui-angular-tutorial.png)

Then, follow the instructions to create a new project and make sure it runs successfully.
https://docs.aspnetzero.com/en/aspnet-core-angular/latest/Getting-Started-Angular

## Install Kendo UI Angular

Let's start by installing a [Calendar](https://www.telerik.com/kendo-angular-ui/components/dateinputs/calendar/) component. Run the following command to install the Kendo UI Angular DateInputs package.

```bash
ng add @progress/kendo-angular-dateinputs 
ng add @progress/kendo-drawing 
ng add @progress/kendo-angular-dialog
```

After the installation, run the following command to creating bundles for the application.

```bash
yarn run create-dynamic-bundles
```

Add the following code to the `app-shared.module.ts` file to import the Kendo UI Angular DateInputs module.

```typescript
import { DateInputsModule } from '@progress/kendo-angular-dateinputs';

@NgModule({
    imports: [
        .
        .
        .
        .
        DateInputsModule,
    ],
})
```

> Don't forget to activate the Kendo UI Angular license. You can find the instructions [here](https://www.telerik.com/kendo-angular-ui/components/my-license/).


## Use Kendo UI Angular Components

First, open the `AppPermissions.cs` file and add the following permission.

```csharp
public const string Pages_Administration_KendoUi = "Pages.Administration.KendoUi";
```

Then, open the `AppAuthorizationProvider.cs` file and add the following permission to the `PermissionChecker` class.

```csharp
var administration = pages.CreateChildPermission(AppPermissions.Pages_Administration, L("Administration"));

administration.CreateChildPermission(AppPermissions.Pages_Administration_KendoUi, L("KendoUi"));
```

Create `kendo-ui` folder in the `src/app/admin` folder and add the following files.

*kendo-ui.component.ts*
```typescript
import { Component, Injector, OnInit } from '@angular/core';
import { appModuleAnimation } from '@shared/animations/routerTransition';
import { AppComponentBase } from '@shared/common/app-component-base';

@Component({
    templateUrl: './kendo-ui.component.html',
    animations: [appModuleAnimation()],
})
export class KendoUiComponent extends AppComponentBase implements OnInit {

    constructor(injector: Injector) {
        super(injector);
    }

    ngOnInit(): void {
    }
}
```

*kendo-ui.component.html*
```html
<div [@routerTransition]>
    <sub-header [title]="'KendoUi' | localize" [description]="'KendoUiHeaderInfo' | localize">
        <div role="actions"> </div>
    </sub-header>
    <div [class]="containerClass">
        <kendo-calendar></kendo-calendar>
    </div>
</div>
```

*kendo-ui-routing.module.ts*
```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { KendoUiComponent } from './kendo-ui.component';

const routes: Routes = [
    {
        path: '',
        component: KendoUiComponent,
        pathMatch: 'full',
    },
];

@NgModule({
    imports: [RouterModule.forChild(routes)],
    exports: [RouterModule],
})
export class KendoUiRoutingModule {}
```

*kendo-ui.module.ts*
```typescript
import { NgModule } from '@angular/core';
import { KendoUiRoutingModule } from './kendo-ui-routing.module';
import { AdminSharedModule } from '@app/admin/shared/admin-shared.module';
import { AppSharedModule } from '@app/shared/app-shared.module';
import { KendoUiComponent } from './kendo-ui.component';

@NgModule({
    declarations: [KendoUiComponent],
    imports: [AppSharedModule, AdminSharedModule, KendoUiRoutingModule],
})
export class KendoUiModule {}
```

Then, open the `admin-routing.module.ts` file and add the following route.

```typescript
{
    path: 'kendo-ui',
    loadChildren: () => import('./kendo-ui/kendo-ui.module').then((m) => m.KendoUiModule),
    data: { permission: 'Pages.Administration.KendoUi' },
},
```

Finally, open the `app-navigation.service.ts` file and add the following menu item.

```typescript
new AppMenuItem(
    'Kendo UI',
    'Pages.Administration.KendoUi',
    'flaticon-calendar',
    '/app/admin/kendo-ui'
),
```

Now, run the application and navigate to the `kendo-ui` page. If you successfully implemented KendoUi, you should see the Kendo UI Angular Calendar component.
![kendo-ui-angular-calendar](images/Blog/kendo-ui-angular-calendar.png)

## Simple CRUD Operations with Kendo UI Angular Grid

### Install Kendo UI Angular Grid

Run the following command to install the Kendo UI Angular Grid package.

```bash
ng add @progress/kendo-angular-grid
```

After the installation, run the following command to creating bundles for the application.

```bash
yarn run create-dynamic-bundles
```

Add the following code to the `app-shared.module.ts` file to import the Kendo UI Angular Grid module.

```typescript
import { GridModule } from '@progress/kendo-angular-grid';

@NgModule({
    imports: [
        .
        .
        .
        .
        GridModule,
    ],
})
```

### Create components for CRUD operations

Let's create a simple CRUD operation using the Kendo UI Angular Grid component. First, create a new folder named `kendo-ui-grid` in the `src/app/admin` folder and add the following files.

Add the `Pages.Administration.KendoUiGrid` permission to the `Host` project, like the first example.

*kendo-ui-grid.component.ts*
```typescript
import { Component, Injector, OnInit } from '@angular/core';
import { appModuleAnimation } from '@shared/animations/routerTransition';
import { AppComponentBase } from '@shared/common/app-component-base';

@Component({
    templateUrl: './kendo-ui-grid.component.html',
    animations: [appModuleAnimation()],
})
export class KendoUiGridComponent extends AppComponentBase implements OnInit {

    constructor(injector: Injector) {
        super(injector);
    }

    ngOnInit(): void {
    }
}
```

*kendo-ui-grid.component.html*
```html
<div [@routerTransition]>
    <sub-header [title]="'KendoUiCrudExample' | localize" [description]="'KendoUiCrudExampleHeaderInfo' | localize">
        <div role="actions"> </div>
    </sub-header>
    <div [class]="containerClass">
        <kendo-grid></kendo-grid>
    </div>
</div>
```

*kendo-ui-grid-routing.module.ts*
```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { KendoUiGridComponent } from './kendo-ui-grid.component';

const routes: Routes = [
    {
        path: '',
        component: KendoUiGridComponent,
        pathMatch: 'full',
    },
];

@NgModule({
    imports: [RouterModule.forChild(routes)],
    exports: [RouterModule],
})
export class KendoUiGridRoutingModule {}
```

*kendo-ui.module.ts*
```typescript
import { NgModule } from '@angular/core';
import { KendoUiGridRoutingModule } from './kendo-ui-grid-routing.module';
import { AdminSharedModule } from '@app/admin/shared/admin-shared.module';
import { AppSharedModule } from '@app/shared/app-shared.module';
import { KendoUiGridComponent } from './kendo-ui-grid.component';

@NgModule({
    declarations: [KendoUiGridComponent],
    imports: [AppSharedModule, AdminSharedModule, KendoUiGridRoutingModule],
})
export class KendoUiGridModule {}
```

Then, open the `admin-routing.module.ts` file and add the following route.

```typescript
{
    path: 'kendo-ui',
    loadChildren: () => import('./kendo-ui/kendo-ui.module').then((m) => m.KendoUiGridModule),
    data: { permission: 'Pages.Administration.KendoUiGrid' },
},
```
Finally, open the `app-navigation.service.ts` file and add the following menu item.

```typescript
new AppMenuItem(
    'Kendo UI CRUD Example',
    'Pages.Administration.KendoUiGrid',
    'flaticon-calendar',
    '/app/admin/kendo-ui-grid'
),
```

Now, run the application and navigate to the `kendo-ui-grid` page. If you successfully implemented KendoUi, you should see the Kendo UI Angular Grid component.

### Host Part of the Application

// TODO: Add the Host Part of the CRUD Application

### Angular Part of the Application 

// TODO: Add the Angular Part of the CRUD Application