# Spinner

We use [ngx-spinner](https://github.com/Napster2210/ngx-spinner) block UI element and loading effect.

#### Enable/Disable FullScreen Spinner

To block screen you can use spinner service.

```typescript
export class DemoUiComponentsComponent extends AppComponentBase implements OnInit {
    constructor(
        injector: Injector
    ) {
        super(injector);
    }
    ngOnInit(): void {
        //show default spinner which cover all page
        this.spinnerService.show();

        setTimeout(() => {
            this.spinnerService.hide();
        }, 1000);
    }
}
```

![infrastructure-angular-spinner-fullscreen](images\infrastructure-angular-spinner-fullscreen.png)



#### Enable/Disable Spinner on Html Element

To use spinner on html element you can use `busyIf` directive.  

The element which has `busyIf` directive will be blocked until input is false.

```html
 <div class="kt-portlet kt-portlet--height-fluid">   
     ...
     <div class="kt-portlet__body">
		<div [busyIf]="loading">
            
         </div>
     </div>
     ...
</div>
```

```typescript
export class MyComponent extends AppComponentBase implements OnInit {
    constructor(
        injector: Injector
    ) {
        super(injector);
    }
    loading = true;
    ngOnInit(): void {
        setTimeout(() => {
            loading = false;
        }, 3000);
    }
}
```

![infrastructure-angular-spinner-on-html](images\infrastructure-angular-spinner-on-html.png)



#### Customize Spinner

Since we use [ngx-spinner](https://github.com/Napster2210/ngx-spinner) you can customize your spinner.

Implementations are located in `busy-if.directive.ts` and `root.component.ts`

*busy-if.directive.ts*

```typescript
@Directive({
    selector: '[busyIf]'
})
export class BusyIfDirective implements OnChanges {
   ...
    loadComponent() {
       ...       
        component.type = "ball-scale-multiple";//you can change fields of component to customize it
        component.size = "medium";
    }
...
}
```

 *root.component.ts*

```typescript
import { Component } from '@angular/core';

@Component({
    selector: 'app-root',
    template: `<router-outlet></router-outlet><ngx-spinner type = "ball-clip-rotate"></ngx-spinner>` //you can customize fullscreen spinner in here
})
export class RootComponent {

}
```



Check **ngx-spinner** documentation for more customization.