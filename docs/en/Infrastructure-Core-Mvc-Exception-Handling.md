# Exception Handling

ASP.NET Zero uses ABP's [exception handling](https://aspnetboilerplate.com/Pages/Documents/AspNet-Core#exception-filter) system. Thus, you don't need to handle & care about exceptions in most time.

ASP.NET Zero solution adds **exception handling middlewares** in the **Startup** class like that:

```csharp
if (env.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseStatusCodePagesWithRedirects("~/Error?statusCode={0}");
    app.UseExceptionHandler("/Error");
}
```

So, you get a nicely formatted exception page in development and a more user friendly error page in production. See **ErrorController** and it's related views (**Views\\Error**) for details.