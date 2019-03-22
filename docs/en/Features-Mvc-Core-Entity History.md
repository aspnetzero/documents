# Entity History

In change logs under audit logs menu, we can see all change logs (entity history) in the application:

<img src="D:/Github/documents/docs/en/images/entity-history-logs.png" alt="Change Logs" class="img-thumbnail" />

When we click the magnifier icon, we can see all details a change log. We can see which properties are changed.

<img src="D:/Github/documents/docs/en/images/entity-history-log-detail.png" alt="Change Log Detail" class="img-thumbnail" />

You should add entity type that you want to track to `*.Core\EntityHistory\EntityHistoryHelper.TrackedTypes`. And make sure you uncomment following line in `*.EntityFrameworkCore\EntityFrameworkCore\YourProjectNameEntityFrameworkCoreModule.cs`

```
//Configuration.EntityHistory.Selectors.Add("AbpZeroTemplateEntities", EntityHistoryHelper.TrackedTypes);
```