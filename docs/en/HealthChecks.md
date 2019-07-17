# Health Checks

Aspnet zero has an implementation of the health checks and health checks ui. 

You can enable or disable health checks and health checks ui.

###### Settings

Health checks settings are located in the `appsettings.json` file

```json
"HealthChecks": {
    "HealthChecksEnabled": true,//enable/disable all health checks.
    "HealthChecksUI": {
      "HealthChecksUIEnabled": true,//enable/disable health checks ui
      "HealthChecks": [
        {
          "Name": "MyCompanyName.AbpZeroTemplate.Web.MVC",//your app name
          "Uri": "http://localhost:62114/healthz"/* your_project_url/healthz
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



> Note: If you enable health checks ui, don't forget to change your `healthz` url before you publish your website.



#### Adding new health check

There are a lot of libraries which you can add to your health check easily.

```
AspNetCore.HealthChecks.System
AspNetCore.HealthChecks.Network
AspNetCore.HealthChecks.SqlServer
...
```

See their own documentation.

###### Adding your custom health check

To add your own health check you should create a class inherited from `IHealthCheck`. ( We locate our health checks `.Application` )

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

In both cases, you should add your health checks to HealthCheckBuilder. Our HealthCheckBuilder is located in `.Web.Core` project.  `( .Web.Core -> HealthCheck -> AbpZeroHealthCheck.cs  -> AddAbpZeroHealthCheck )` .  

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

After adding your new health check here, you will be able to see its status in json and ui automatically.

------

**Endpoint:**

- *MyCompanyName.AbpZeroTemplate.Web.Mvc*

  Health checks ui endpoint: http://localhost:62114/healthchecks-ui   (if it is enabled)

  Health checks json result endpoint: http://localhost:62114/healthz  (if it is enabled)

- *MyCompanyName.AbpZeroTemplate.Web.Host (Angular projects can use that health check)*

  Health checks ui endpoint: http://localhost:22742/healthchecks-ui   (if it is enabled)

  Health checks json result endpoint: http://localhost:22742/healthz  (if it is enabled)

- MyCompanyName.AbpZeroTemplate.Web.Public

  Health checks ui endpoint: http://localhost:45776/healthchecks-ui   (if it is enabled)

  Health checks json result endpoint: http://localhost:45776/healthz  (if it is enabled)


see also:  
https://github.com/xabaril/AspNetCore.Diagnostics.HealthChecks

https://docs.docker.com/engine/reference/builder/#healthcheck
            
