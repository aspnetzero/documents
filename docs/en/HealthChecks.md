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
          "Uri": "http://localhost:62114/healthz"//you should change your url before you publish project
        }
      ],
      "EvaluationTimeOnSeconds": 10,
      "MinimumSecondsBetweenFailureNotifications": 60
    }
  }
```



> Note: If you enable health checks ui, don't forget to change your `healthz` url before you publish your website.



#### Adding new health check

There are a lot of libraries which you can add to your health check easily.

```
Install-Package AspNetCore.HealthChecks.System
Install-Package AspNetCore.HealthChecks.Network
Install-Package AspNetCore.HealthChecks.SqlServer
Install-Package AspNetCore.HealthChecks.MongoDb
Install-Package AspNetCore.HealthChecks.Npgsql
Install-Package AspNetCore.HealthChecks.Elasticsearch
Install-Package AspNetCore.HealthChecks.Redis
Install-Package AspNetCore.HealthChecks.EventStore
Install-Package AspNetCore.HealthChecks.AzureStorage
Install-Package AspNetCore.HealthChecks.AzureServiceBus
Install-Package AspNetCore.HealthChecks.AzureKeyVault
Install-Package AspNetCore.HealthChecks.MySql
Install-Package AspNetCore.HealthChecks.DocumentDb
Install-Package AspNetCore.HealthChecks.SqLite
Install-Package AspNetCore.HealthChecks.RavenDB
Install-Package AspNetCore.HealthChecks.Kafka
Install-Package AspNetCore.HealthChecks.RabbitMQ
Install-Package AspNetCore.HealthChecks.OpenIdConnectServer
Install-Package AspNetCore.HealthChecks.DynamoDB
Install-Package AspNetCore.HealthChecks.Oracle
Install-Package AspNetCore.HealthChecks.Uris
Install-Package AspNetCore.HealthChecks.Aws.S3
Install-Package AspNetCore.HealthChecks.Consul
Install-Package AspNetCore.HealthChecks.Hangfire
Install-Package AspNetCore.HealthChecks.SignalR
Install-Package AspNetCore.HealthChecks.Kubernetes
Install-Package AspNetCore.HealthChecks.Gcp.CloudFirestore
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



Health checks ui endpoint is: http://localhost:62114/healthchecks-ui   (if it is enabled)

Health checks json result endpoint is: http://localhost:62114/healthz  (if it is enabled)

see also: https://github.com/xabaril/AspNetCore.Diagnostics.HealthChecks

​		https://docs.docker.com/engine/reference/builder/#healthcheck


​                