# Create Custom Input Types

Dynamic property system comes with 4 built-in input types to meet most needs:

- **SINGLE_LINE_STRING** - Simple text input
- **CHECKBOX** - Boolean checkbox
- **COMBOBOX** - Single-select dropdown
- **MULTISELECTCOMBOBOX** - Multi-select dropdown

But you can also create your own input types as shown below.

## Backend: Register the Input Type in Core

1. Go to `*.Core` and create a folder named `CustomInputTypes`.

2. Create a class named `UserSelectInputType` in that folder.

   ```csharp
   /// <summary>
   /// User Select value UI type.
   /// </summary>
   [Serializable]
   [InputType("USERSELECT")]
   public class UserSelectInputType : InputTypeBase
   { 
   }
   ```

3. Go to `AppDynamicEntityParameterDefinitionProvider` and add new input type.

   ```csharp
   public class AppDynamicEntityParameterDefinitionProvider : DynamicEntityParameterDefinitionProvider
   {
       public override void SetDynamicEntityParameters(IDynamicEntityParameterDefinitionContext context)
       {
           // ... existing code
           context.Manager.AddAllowedInputType<UserSelectInputType>();
       }
   }
   ```

## Frontend: Understanding the Input Type System

Custom input types in React are implemented as functional components that implement the `InputTypeComponentProps` interface. All input type components are located in:

```
src/pages/admin/components/common/input-types/
├── input-components/          # Input component implementations
│   ├── SingleLineStringInput.tsx
│   ├── CheckboxInput.tsx
│   ├── ComboboxInput.tsx
│   └── MultiSelectComboboxInput.tsx
├── input-types.constants.ts   # Input type registry
└── types.ts                   # TypeScript interfaces
```

## Frontend: Creating a Custom Input Component

First, understand the interface your component must implement. The `InputTypeComponentProps` interface is defined in [types.ts](../src/pages/admin/components/common/input-types/types.ts):

```typescript
export interface InputTypeComponentProps {
  selectedValues: string[];
  allValues: string[];
  onChange: (values: string[]) => void;
  onInstance?: (instance: unknown) => void;
}
```

- **selectedValues**: The currently selected value(s) as a string array
- **allValues**: Available options (for dropdown-type inputs)
- **onChange**: Callback to notify parent of value changes
- **onInstance**: Optional callback to expose component methods to parent

### Sample Component: User Autocomplete Input

Create a new file at `src/pages/admin/components/common/input-types/input-components/UserAutocompleteInput.tsx`:

```tsx
import React, { useEffect, useRef, useState } from "react";
import { AutoComplete } from "antd";
import { NameValueDto } from "@/api/generated/api";
import { useServiceProxy } from "@/hooks/useServiceProxy";
import type { InputTypeComponentProps } from "../types";

const UserAutocompleteInput: React.FC<InputTypeComponentProps> = ({
  selectedValues,
  onChange,
  onInstance,
}) => {
  const [value, setValue] = useState<string>(selectedValues?.[0] ?? "");
  const [options, setOptions] = useState<{ label: string; value: string }[]>([]);
  const [selectedUser, setSelectedUser] = useState<NameValueDto | null>(null);
  const ref = useRef<{ getSelectedValues: () => string[] } | null>(null);
  
  const { commonLookupService } = useServiceProxy();

  // Sync with external selectedValues
  useEffect(() => {
    setValue(selectedValues?.[0] ?? "");
  }, [selectedValues]);

  // Expose getSelectedValues method to parent
  useEffect(() => {
    ref.current = {
      getSelectedValues: () => (selectedUser?.value ? [selectedUser.value] : []),
    };
    if (onInstance) onInstance(ref.current);
  }, [selectedUser, onInstance]);

  // Load initial user if selectedValue is provided
  useEffect(() => {
    if (selectedValues?.[0] && !selectedUser) {
      commonLookupService
        .findUsers({
          filterText: "",
          maxResultCount: 1,
          excludeCurrentUser: false,
        })
        .then((result) => {
          if (result.items && result.items.length === 1) {
            setSelectedUser(result.items[0]);
            setValue(result.items[0].name ?? "");
          }
        });
    }
  }, [selectedValues, selectedUser, commonLookupService]);

  const handleSearch = async (searchText: string) => {
    if (!searchText) {
      setOptions([]);
      return;
    }

    const result = await commonLookupService.findUsers({
      filterText: searchText,
      maxResultCount: 50,
      excludeCurrentUser: false,
    });

    setOptions(
      (result.items ?? []).map((user) => ({
        label: user.name ?? "",
        value: user.value ?? "",
      }))
    );
  };

  const handleSelect = (selectedValue: string, option: { label: string; value: string }) => {
    const user: NameValueDto = { name: option.label, value: selectedValue };
    setSelectedUser(user);
    setValue(option.label);
    onChange([selectedValue]);
  };

  return (
    <AutoComplete
      value={value}
      options={options}
      onSearch={handleSearch}
      onSelect={handleSelect}
      onChange={(text) => setValue(text)}
      placeholder="Search users..."
      style={{ width: "100%" }}
    />
  );
};

export default UserAutocompleteInput;
```

## Frontend: Registering the Custom Input Type

After creating the input component, you need to register it in the input types configuration.

### Step 1: Add the Type Name

Open [types.ts](../src/pages/admin/components/common/input-types/types.ts) and add your new type to the `InputTypeName` union:

```typescript
export type InputTypeName =
  | "SINGLE_LINE_STRING"
  | "CHECKBOX"
  | "COMBOBOX"
  | "MULTISELECTCOMBOBOX"
  | "USERSELECT"; // Add your custom type
```

### Step 2: Register in Constants

Open [input-types.constants.ts](../src/pages/admin/components/common/input-types/input-types.constants.ts) and add your component:

```typescript
import type { InputTypeConfigurationDefinition } from "./types";
import SingleLineStringInput from "./input-components/SingleLineStringInput";
import CheckboxInput from "./input-components/CheckboxInput";
import ComboboxInput from "./input-components/ComboboxInput";
import MultiSelectComboboxInput from "./input-components/MultiSelectComboboxInput";
import UserAutocompleteInput from "./input-components/UserAutocompleteInput";

export const INPUT_TYPES = {
  SINGLE_LINE_STRING: "SINGLE_LINE_STRING",
  CHECKBOX: "CHECKBOX",
  COMBOBOX: "COMBOBOX",
  MULTISELECTCOMBOBOX: "MULTISELECTCOMBOBOX",
  USERSELECT: "USERSELECT", // Add the constant
} as const;

export const InputTypeConfigurationDefinitions: InputTypeConfigurationDefinition[] = [
  {
    name: "SINGLE_LINE_STRING",
    component: SingleLineStringInput,
    hasValues: false,
  },
  { name: "CHECKBOX", component: CheckboxInput, hasValues: false },
  { name: "COMBOBOX", component: ComboboxInput, hasValues: true },
  {
    name: "MULTISELECTCOMBOBOX",
    component: MultiSelectComboboxInput,
    hasValues: true,
  },
  {
    name: "USERSELECT",
    component: UserAutocompleteInput,
    hasValues: false, // false because values come from API, not pre-defined
  },
];
```

The `hasValues` property indicates whether the input type requires a pre-defined list of values (like combobox options) or fetches/generates values on its own.

## Using the Custom Input Type

That's all! Your new custom input type is ready to use. Navigate to the Dynamic Entity Properties page and create a new entity property. You will see your `USERSELECT` input type in the dropdown and can use it for any entity.

## Input Type Configuration Summary

| Property | Description |
|----------|-------------|
| `name` | Unique identifier for the input type (must match backend `InputType` attribute and `InputTypeName` union) |
| `component` | React component that implements `InputTypeComponentProps` |
| `hasValues` | Whether the input requires pre-defined values (shown in admin UI) |

## Best Practices

1. **Always implement `getSelectedValues`** - Expose this method via `onInstance` so the parent can retrieve values programmatically.

2. **Sync with `selectedValues` prop** - Use `useEffect` to update internal state when the prop changes.

3. **Call `onChange` on value changes** - Notify the parent component whenever the selected value changes.

4. **Return values as string array** - All input types must return their values as `string[]`, even for single-value inputs.

5. **Match backend input type name** - The `name` in `InputTypeConfigurationDefinitions` must exactly match the `[InputType("NAME")]` attribute on your backend class.

