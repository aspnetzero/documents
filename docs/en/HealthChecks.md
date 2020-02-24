# Health Checks

AspNet Zero has an implementation of health checks and health checks UI. 

You can enable or disable health checks and health checks UI.

###### Settings

Health checks settings are located in the `appsettings.json` file

```json
"HealthChecks": {
    "HealthChecksEnabled": true, //enable/disable all health checks.
    "HealthChecksUI": {
      "HealthChecksUIEnabled": true, //enable/disable health checks ui
      "HealthChecks": [
        {
          "Name": "MyCompanyName.AbpZeroTemplate.Web.MVC", //your app name
          "Uri": "https://localhost:44302/health" /* your_project_url/health
			you should change that url before you publish your project*/
        }
      ],
      "EvaluationTimeOnSeconds": 10,
      "MinimumSecondsBetweenFailureNotifications": 60,
      //"HealthCheckDatabaseConnectionString": "Data Source=[PUT-MY-PATH-HERE]\\healthchecksdb" //-> Optional, default on WebContentRoot,
      //for example, if you use azure you may need to set this connection string
    }
  }
```



> Note: If you enable Health Checks UI, don't forget to change your `health` URL before you publish your website.



#### Adding new health check

There are a lot of libraries which you can add to your health check easily. To see a full list of libraries and used package in AspNet Zero, see [https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks). Here are some sample package names:

```
AspNetCore.HealthChecks.System
AspNetCore.HealthChecks.Network
AspNetCore.HealthChecks.SqlServer
...
```

###### Adding your custom health check

To add your own health check, you should create a class inherited from `IHealthCheck`. ( Already implemented health check classes are located under **HealthChecks** folder of `.Application`  project.)

```c#
  public class MyCustomHealthCheck : IHealthCheck
    {
        private readonly MyService _myService;
        public AbpZeroTemplateDbContextHealthCheck(MyService myService)
        {
            _myService = myService;
        }

        public Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = new CancellationToken())
        {
            if (_myService.CheckSomeThing())
            {
                return Task.FromResult(HealthCheckResult.Healthy("MyService is healthy."));
            }

            return Task.FromResult(HealthCheckResult.Unhealthy("MyService is unhealthy."));
        }
    }
```

In both cases, you should add your health checks to **HealthCheckBuilder**. HealthCheckBuilder is located in `.Web.Core` project.  `( .Web.Core -> HealthCheck -> AbpZeroHealthCheck.cs  -> AddAbpZeroHealthCheck )` .  

```c#
public static class AbpZeroHealthCheck
    {
        public static IHealthChecksBuilder AddAbpZeroHealthCheck(this IServiceCollection services)
        {
            var builder = services.AddHealthChecks();
            builder.AddCheck<AbpZeroTemplateDbContextHealthCheck>("Database Connection");
            builder.AddCheck<AbpZeroTemplateDbContextUsersHealthCheck>("Database Connection with user check");
            builder.AddCheck<CacheHealthCheck>("Cache");

            // add your other health checks here
            // builder.AddCheck<MyCustomHealthCheck>("my health check");
            //...
            return builder;
        }
    }
```

After adding your new health check here, you will be able to see its status in JSON and UI automatically.

------

**Endpoints:**

- *MVC project (Only exists in ASP.NET Core & jQuery version)*

  Health checks UI endpoint: https://localhost:44302/healthchecks-ui   (if it is enabled)

  Health checks JSON result endpoint: https://localhost:44302/health  (if it is enabled)

- *Host project (Available in ASP.NET Core versions but designed for Angular project)*

  Health checks UI endpoint: https://localhost:44301/healthchecks-ui   (if it is enabled)

  Health checks JSON result endpoint: https://localhost:44301/health  (if it is enabled)

- *Public Website*

  Health checks UI endpoint: http://localhost:45776/healthchecks-ui   (if it is enabled)

  Health checks JSON result endpoint: http://localhost:45776/health  (if it is enabled)

see also:  
https://github.com/xabaril/AspNetCore.Diagnostics.HealthChecks

https://docs.docker.com/engine/reference/builder/#healthcheck
           

> Note: If you enable health checks, it will create its dbContext. That's why, when you try to create new migration or update database you should also use `-c [YourProjectName]DbContext` command.