# Creating the PhoneBook Component

Create a **phonebook** folder inside **src/app/main** folder and add a
new typescript file (**phonebook.component.ts**) in the phonebook folder
as shown below:

```javascript
import { Component } from '@angular/core';
import { AppComponentBase } from '@shared/common/app-component-base';
import { appModuleAnimation } from '@shared/animations/routerTransition';
import { LocalizePipe } from '@shared/common/pipes/localize.pipe';
import { SubHeaderComponent } from '@app/shared/common/sub-header/sub-header.component';

@Component({
    templateUrl: './phonebook.component.html',
    animations: [appModuleAnimation()],
    imports: [SubHeaderComponent, LocalizePipe],
})
export class PhoneBookComponent extends AppComponentBase {
    constructor() {
        super();
    }
}
```

We inherited from **AppComponentBase** which provides some common
functions and fields (like localization and access control) for us. It's
not required, but makes our job easier.

As we declared in **phonebook.component.ts** we should create a
**phonebook.component.html** view in the same phonebook folder:

```html
<div [@routerTransition]>
    <sub-header [title]="'PhoneBook' | localize" [description]="'PhoneBooksHeaderInfo' | localize"></sub-header>
    <div [class]="containerClass">
        <div class="card card-custom">
            <div class="card-body">
                <p>PHONE BOOK CONTENT COMES HERE!</p>
            </div>
        </div>
    </div>
</div>
```

**l** (lower case 'L') function comes from **AppComponentBase** and used
to easily localize texts. **\[@routerTransition\]** attribute is
required for page transition animation.

Now we should create a **phonebook.module.ts** and **phonebook-routing.module.ts**  view in the same phonebook folder:

*phonebook-routing.module.ts*

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
    {
        path: '',
        loadComponent: () => import('./phonebook.component').then((m) => m.PhoneBookComponent),
        pathMatch: 'full',
    },
];

@NgModule({
    imports: [RouterModule.forChild(routes)],
    exports: [RouterModule],
})
export class PhoneBookRoutingModule {}
```

*phonebook.module.ts*

```typescript
import { NgModule } from '@angular/core';
import { AppSharedModule } from '@app/shared/app-shared.module';
import { PhoneBookRoutingModule } from './phonebook-routing.module';
import { PhoneBookComponent } from './phonebook.component';

@NgModule({
    imports: [AppSharedModule, PhoneBookRoutingModule, PhoneBookComponent],
})
export class PhoneBookModule {}
```

Now, we can refresh the page to see the new added page:

<img src="images/phonebook-empty-ng3.png" alt="Phonebook empty" class="img-thumbnail" style="width:100.0%" />

Note: Angular-cli automatically re-compiles and refreshes the page when
any changes made to any file in the application.

## Next

- [Creating Person Entity](Developing-Step-By-Step-Angular-Creating-Person-Entity)
