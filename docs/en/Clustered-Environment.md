# Deploying to a Clustered Environment

This document introduces the topics that you should consider when you are deploying your application to a clustered environment where **multiple instances of your application run concurrently**, and explains how you can deal with these topics in your ASP.NET Boilerplate or ASP.NET Zero based application.


## Understanding the Clustered Environment

> You can skip this section if you are already familiar with clustered deployment and load balancers.

### Single Instance Deployment

Consider an application deployed as a **single instance**, as illustrated in the following figure:

![deployment-single-instance](./images/deployment-single-instance.png)

Browsers and other client applications can directly make HTTP requests to your application. You can put a web server (e.g. IIS or NGINX) between the clients and your application, but you still have a single application instance running in a single server or container. Single-instance configuration is **limited to scale** since it runs in a single server and you are limited with the server's capacity.

### Clustered Deployment

**Clustered deployment** is the way of running **multiple instances** of your application **concurrently** in a single or multiple servers. In this way, different instances can serve different requests and you can scale by adding new servers to the system. The following figure shows a typical implementation of clustering using a **load balancer**:

![deployment-clustered](./images/deployment-clustered.png)

### Load Balancers

[Load balancers](https://en.wikipedia.org/wiki/Load_balancing_(computing)) have a lot of features, but they fundamentally **forward an incoming HTTP request** to an instance of your application and return your response back to the client application.

Load balancers can use different algorithms for selecting the application instance while determining the application instance that is used to deliver the incoming request. **Round Robin** is one of the simplest and most used algorithms. Requests are delivered to the application instances in rotation. First instance gets the first request, second instance gets the second, and so on. It returns to the first instance after all the instances are used, and the algorithm goes like that for the next requests.

### Potential Problems

Once multiple instances of your application run in parallel, you should carefully consider the following topics:

* Any **state (data) stored in memory** of your application will become a problem when you have multiple instances. A state stored in memory of an application instance may not be available in the next request since the next request will be handled by a different application instance. While there are some solutions (like sticky sessions) to overcome this problem user-basis, it is a **best practice to design your application as stateless** if you want to run it in a cluster, container or/and cloud.
* **In-memory caching** is a kind of in-memory state and should not be used in a clustered application. You should use **distributed caching** instead.
* You shouldn't store data in the **local file system**. It should be available to all instances of your application. Different application instance may run in different containers or servers and they may not be able to have access to the same file system. You can use a **cloud or external storage provider** as a solution.
* If you have **background workers** or **job queue managers**, you should be careful since multiple instances may try to execute the same job or perform the same work concurrently. As a result, you may have the same work done multiple times or you may get a lot of errors while trying to access and change the same resources.

## Switching to a Distributed Cache

ASP.NET Core provides different kind of caching features. [In-memory cache](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/memory) stores your objects in the memory of the local server and is only available to the application that stored the object. Non-sticky sessions in a clustered environment should use the [distributed caching](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed) except some specific scenarios (for example, you can cache a local CSS file into memory. It is read-only data and it is the same in all application instances. You can cache it in memory for performance reasons without any problem).

[ASPNET Boilerplate caching system](https://aspnetboilerplate.com/Pages/Documents/Caching) extends [ASP.NET Core's in-memory cache](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/memory?view=aspnetcore-6.0) infrastructure. It works in-memory by default. You should configure an actual distributed cache provider when you want to deploy your application to a clustered environment. ASPNET Boilerplate provides built-in Redis implementation. It is already implemented in ASPNET Zero. You should go to the `[YOURAPPNAME]WebCoreModule` and uncomment following code.

_[YOURAPPNAME]WebCoreModule.cs_
```csharp
Configuration.Caching.UseRedis(options =>
{
    options.ConnectionString = _appConfiguration["Abp:RedisCache:ConnectionString"];
    options.DatabaseId = _appConfiguration.GetValue<int>("Abp:RedisCache:DatabaseId");
});
```

For more information check [caching](https://aspnetboilerplate.com/Pages/Documents/Caching#redis-cache-integration) documentation to enable redis cache. 

## Configuring Background Jobs

ASPNET Boilerplate's [background job system](https://aspnetboilerplate.com/Pages/Documents/Background-Jobs-And-Workers) is used to queue tasks to be executed in the background. Background job queue is persistent and a queued task is guaranteed to be executed (it is re-tried if it fails).

If you want to run background jobs in multiple instances ASPNET Boilerplates provides you Hangfire implementation. You can replace background job system with Hangfire. It is already implemented in ASPNET Zero. To enable Hangfire, go to the `WebConsts.cs` and set `HangfireDashboardEnabled` true.

_WebConsts.cs_
```csharp
namespace MyCompanyName.AbpZeroTemplate.Web.Common
{
    public static class WebConsts
    {
        //...
        public static bool HangfireDashboardEnabled = true;
```

For more information check [Hangfire integration](https://docs.aspnetzero.com/en/aspnet-core-mvc/latest/Infrastructure-Background-Jobs) documentation.

If you don't want to run background jobs in multiple instances, you may stop the background job manager (set `Configuration.BackgroundJobs.IsJobExecutionEnabled` to `false`) in all application instances except one of them, so only the single instance executes the jobs (while other application instances can still queue jobs).

```csharp
public class MyProjectWebModule : AbpModule
{
    public override void PreInitialize()
    {
        Configuration.BackgroundJobs.IsJobExecutionEnabled = false;
    }

    //...
}
```

For more information check [background jobs and workers](https://aspnetboilerplate.com/Pages/Documents/Background-Jobs-And-Workers) documentation.

## Scaling SignalR

ASPNET Zero has a built-in chat system and real-time notification system. They both use [signalR](https://docs.microsoft.com/en-us/aspnet/core/signalr/introduction). Before using multiple instances of your project, you should check [signalR's scaling documentation](https://docs.microsoft.com/en-us/aspnet/core/signalr/scale) and choose SignalR backplane providers or AzureSignalR. 