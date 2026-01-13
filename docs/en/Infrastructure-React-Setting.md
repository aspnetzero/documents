# Setting

You can use the global `abp.setting` object to get setting values in your React components.

## Usage

```tsx
// Get a setting value
const settingValue = abp.setting.get("SettingName");

// Get a boolean setting
const boolValue = abp.setting.getBoolean("SettingName");

// Get an integer setting
const intValue = abp.setting.getInt("SettingName");
```

Settings are defined on the server:

* [Host Settings](Features-React-Host-Settings)
* [Tenant Settings](Features-React-Tenant-Settings)

See setting management [documentation](https://aspnetboilerplate.com/Pages/Documents/Setting-Management) for more.


