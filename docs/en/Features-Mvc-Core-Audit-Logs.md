# Audit Logs

In audit logs page, we can see all user interactions with the application:

<img src="images/audit-logs-core-3.png" alt="Audit logs" class="img-thumbnail" />

All application service methods and MVC controller actions are automatically logged and can be viewed here. See [audit logs documentation](https://aspnetboilerplate.com/Pages/Documents/Audit-Logging) to learn how to configure it. When we click the magnifier icon, we can
see all details an audit log:

<img src="images/audit-logs-detail-1.png" alt="Audit Log" class="img-thumbnail" />

Audit log report is provided by **AuditLogAppService** class.

### Periodic Log Deletion 

ASP.NET Zero has built-in periodic log deletion system (`*.Application/Auditing/ExpiredAuditLogDeleterWorker.cs`). To enable it, go to `appsettings.json` and set `AuditLog.AutoDeleteExpiredLogs.IsEnabled` to true; (default `false`)

```json
"App": {
    "AuditLog": {
      "AutoDeleteExpiredLogs": {
        "IsEnabled": true
      }
    }
}
```

Then periodic log deletion will be enabled.

#### Periodic Log Deletion Backup

Periodic log deletion system also has backup implementation. It uses `IExpiredAndDeletedAuditLogBackupService` to backup deleted items. It's default implementation uses excel to create backup. To enable it, go to `appsettings.json` and set `AuditLog.AutoDeleteExpiredLogs.ExcelBackup.IsEnabled` to true; (default `false`). Then deleted items will be stored in the given file path as an excel file.

```json
"App": {
    "AuditLog": {
      "AutoDeleteExpiredLogs": {
        "IsEnabled": true,
        "ExcelBackup": {
          "IsEnabled": true,
          "FilePath": "App_Data/AuditLogsBackups/"
        }
      }
    }
}
```

________


`*.Application/Auditing/ExpiredAuditLogDeleterWorker.cs` has two more parameter.

**CheckPeriodAsMilliseconds:** Time to wait between two controls.

**MaxDeletionCount:** The maximum number of records that can be deleted at once.

> Note: To perform smaller operations with more frequent intervals you can decrease `MaxDeletionCount` and `CheckPeriodAsMilliseconds`. 

## Next

- [Entity History](Features-Mvc-Core-Entity-History)
