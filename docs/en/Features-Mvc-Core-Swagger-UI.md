# Swagger UI

[Swagger UI](http://swagger.io/swagger-ui/) is **integrated** to ASP.NET Zero but **disabled by default**. Swagger UI configuration is located in
**S**<span class="auto-style3">tartup</span> class in the **.Web** project. You can enable it by just uncommenting the related lines:

In Startup.**ConfigureServices** method, enable the following line:

```
services.AddSwaggerGen();
```

And in Startup.Configure method, enable the following lines:

```
app.UseSwagger();
app.UseSwaggerUi();
```

You can browse **swagger ui** with this URL: "/**swagger/ui**".

<img src="images/swagger-ui-core.png" alt="Swagger UI" width="974" height="753" />

Thus, anyone (or any application) can explore, test and use the API easily.

In order to enable comments on Swagger UI, set ```Swagger:ShowSummaries``` to true in appsettings.json. After that, Swagger UI will show summaries written on your application services and controllers.