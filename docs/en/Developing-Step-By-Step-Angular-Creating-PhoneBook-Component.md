# Creating the PhoneBook Component

Create a **phonebook** folder inside **src/app/main** folder and add a
new typescript file (**phonebook.component.ts**) in the phonebook folder
as shown below:

```javascript
import { Component, Injector } from '@angular/core';
import { AppComponentBase } from '@shared/common/app-component-base';
import { appModuleAnimation } from '@shared/animations/routerTransition';

@Component({
    templateUrl: './phonebook.component.html',
    animations: [appModuleAnimation()]
})

export class PhoneBookComponent extends AppComponentBase {
    constructor(
        injector: Injector
    ) {
        super(injector);
    }
}
```

We inherited from **AppComponentBase** which provides some common
functions and fields (like localization and access control) for us. It's
not required, but makes our job easier. Now, we can return back to
**main-routing.module.ts** and add import statement for the newly
created PhoneBookComponent class:

```javascript
import { PhoneBookComponent } from './phonebook/phonebook.component';
```

As we declared in **phonebook.component.ts** we should create a
**phonebook.component.html** view in the same phonebook folder:

```html
<div [@routerTransition]>
    <div class="kt-content  kt-grid__item kt-grid__item--fluid kt-grid kt-grid--hor">
        <div class="kt-subheader kt-grid__item">
            <div [class]="containerClass">
                <div class="kt-subheader__main">
                    <h3 class="kt-subheader__title">
                        <span>{{"PhoneBook" | localize}}</span>
                    </h3>
                </div>
            </div>
        </div>
        <div [class]="containerClass + ' kt-grid__item kt-grid__item--fluid'"
            <div class="kt-portlet kt-portlet--mobile">
                <div class="kt-portlet__body  kt-portlet__body--fit">
                    <p>PHONE BOOK CONTENT COMES HERE!</p>
                </div>
            </div>
        </div>
    </div>
</div>
```

**l** (lower case 'L') function comes from **AppComponentBase** and used
to easily localize texts. **\[@routerTransition\]** attribute is
required for page transition animation.

And finally, Angular requires to relate every component to a **module**.
So, we are making the following changes on the **main.module.ts**:

```javascript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';

import { DashboardComponent } from './dashboard/dashboard.component';
import { PhoneBookComponent } from './phonebook/phonebook.component';

import { ModalModule, TabsModule, TooltipModule } from 'ngx-bootstrap';
import { AppCommonModule } from '@app/shared/common/app-common.module';
import { UtilsModule } from '@shared/utils/utils.module';
import { MainRoutingModule } from './main-routing.module';

@NgModule({
    imports: [
        CommonModule,
        FormsModule,
        ModalModule,
        TabsModule,
        TooltipModule,
        AppCommonModule,
        UtilsModule,
        MainRoutingModule
    ],
    declarations: [
      DashboardComponent,
      PhoneBookComponent
    ]
})
export class MainModule { }
```

Added an **import** statement and added PhoneBookComponent to
**declarations** array. Now, we can refresh the page to see the new
added page:

<img src="images/phonebook-empty-ng2.png" alt="Phonebook empty" class="img-thumbnail" style="width:100.0%" />

Note: Angular-cli automatically re-compiles and refreshes the page when
any changes made to any file in the application.

## Next

- [Creating Person Entity](Developing-Step-By-Step-Angular-Creating-Person-Entity)
