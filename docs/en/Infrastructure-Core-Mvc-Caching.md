# Caching And Redis Cache

ASP.NET Zero uses **in-memory** caching but it's ready to use **Redis** as cache server. If you want to enable it, just uncomment the following line in your **WebCoreModule** (in your .Web.Core project):

```
Configuration.Caching.UseRedis(...);
```

Redis server should be up & running to be able to use it. See [caching documentation](https://aspnetboilerplate.com/Pages/Documents/Caching) for more information.

