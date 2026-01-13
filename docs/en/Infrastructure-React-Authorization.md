# Authorization

You can use the `usePermissions` hook to check user permissions in React components. This hook is located at [src/hooks/usePermissions.ts](../src/hooks/usePermissions.ts).

## Usage

```tsx
import { usePermissions } from "@/hooks/usePermissions";

const MyComponent: React.FC = () => {
  const { isGranted, isGrantedAny } = usePermissions();

  return (
    <div>
      {isGranted("Pages.Administration.Users") && (
        <button>Manage Users</button>
      )}
      
      {isGrantedAny("Pages.Administration.Users.Create", "Pages.Administration.Users.Edit") && (
        <button>Create or Edit User</button>
      )}
    </div>
  );
};
```

## Available Methods

| Method | Description |
|--------|-------------|
| `isGranted(permissionName)` | Returns `true` if the user has the specified permission |
| `isGrantedAny(...permissionNames)` | Returns `true` if the user has any of the specified permissions |

## Permissions Definition

Permissions are defined on the backend in the `*.Core` project. See:
- [Role Management](Features-React-Role-Management)
- [User Management](Features-React-User-Management)
- [ASP.NET Boilerplate authorization](https://aspnetboilerplate.com/Pages/Documents/Authorization) documentation


