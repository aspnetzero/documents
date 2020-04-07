# Create Custom Input Types

In this document we will create a custom input type step by step. Our input type is multi-select combobox input type.

1. Go to `*.Core` and create a folder named `CustomInputTypes`.

2. Create a class named `MultiSelectComboboxInputType` in that folder.

   ```csharp
   /// <summary>
   /// Multi Select Combobox value UI type.
   /// </summary>
   [Serializable]
   [InputType("MULTISELECTCOMBOBOX")]
   public class MultiSelectComboboxInputType : InputTypeBase
   { 
   }
   ```

3. Go to `AppDynamicEntityParameterDefinitionProvider` and add new input type.

   ```csharp
    public class AppDynamicEntityParameterDefinitionProvider : DynamicEntityParameterDefinitionProvider
   {
       public override void SetDynamicEntityParameters(IDynamicEntityParameterDefinitionContext context)
       {
          	...
           context.Manager.AddAllowedInputType<MultiSelectComboboxInputType>();
   		...
       }
   }
   ```

4. Go to `angular\src\app\shared\common\input-types` folder

5. Create new component  named `MultiSelectComboboxInputTypeComponent` as seen below.

   ```bash
   ng g component multi-select-combobox-input-type
   ```

   *multi-select-combobox-input-type.component.html*

   ```typescript
   import { Component, OnInit, Injector } from '@angular/core';
   import { InputTypeComponentBase } from '../input-type-component-base';
   
   @Component({
     templateUrl: './multi-select-combobox-input-type.component.html'
   })
   export class MultiSelectComboboxInputTypeComponent extends InputTypeComponentBase implements OnInit {
     filteredValues: string[];
   
     constructor(
       injector: Injector,
     ) {
       super(injector);
     }
   
     ngOnInit() {
       this.filteredValues = this.allValues;
     }
   
     getSelectedValues(): string[] {
       debugger;
       if (!this.selectedValues) {
         return [];
       }
       return this.selectedValues;
     }
   
     filter(event) {
       this.filteredValues = this.allValues
         .filter(item =>
           item.toLowerCase().includes(event.query.toLowerCase())
         );
     }
   }
   ```

   *multi-select-combobox-input-type.component.html*

   ```html
   <p-autoComplete [(ngModel)]="selectedValues" [suggestions]="filteredValues" (completeMethod)="filter($event)" [minLength]="1" [multiple]="true" inputStyleClass="form-control" styleClass="w-100">
   </p-autoComplete>
   ```

   You must extend  `InputTypeComponentBase`. Since you extend `InputTypeComponentBase` your component will have **selectedValues**(initial stored selected values), **allValues**(all values that your component can have, if your component needs initial values.)

   

6. Then go to `angular\src\app\shared\common\input-types\input-type-configuration.service.ts` and add your input type.

   ```typescript
   export class InputTypeConfigurationService {
     ...
     private initialize(): void {  
     ...
   
       let multiselectComboBoxInputType = new InputTypeConfigurationDefinition(
         'MULTISELECTCOMBOBOX',
         MultiSelectComboboxInputTypeComponent,
         true//is that input type need values to work. For example dropdown need initial values to select.
       );
   
       this.InputTypeConfigurationDefinitions.push(multiselectComboBoxInputType);
     }
     ...
   }
   
   
   ```

7. Go to `angular\src\app\shared\common\app-common.module.ts` and add your component to entryComponents:

   ```typescript
   @NgModule({
     	...
       declarations: [
         	...
           MultipleSelectComboboxInputTypeComponent
       ],
       ...,    
       entryComponents: [
           ...
           MultipleSelectComboboxInputTypeComponent
       ]
   })
   ```

All done. Your custom input type is ready to use in dynamic parameter. Create new dynamic parameter which uses that input type, add it to an entity. Then you can go to manage page and use it. 

![custom-input-type-multi-select-combobox-mvc](images/custom-input-type-multi-select-combobox-angular.png)
