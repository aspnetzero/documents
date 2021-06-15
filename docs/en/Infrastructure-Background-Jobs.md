# Hangfire Integration

[Hangfire](https://www.hangfire.io/) is a library which allows performing background processing in .NET and .NET Core applications. ASP.NET Zero includes Hangfire NuGet packages but it is disabled by default. By default, ASP.NET Zero uses its own background job system, see its [documentation](https://aspnetboilerplate.com/Pages/Documents/Background-Jobs-And-Workers).

If the default background job system doesn't fir for your requirements, you can enable Hangfire by setting ```WebConsts.HangfireDashboardEnabled``` to true in **Common\WebConsts.cs** under your Web.Core project.

Hangfire can be useful when running more than one instance of your web app in production. In such a scenario, Hangfire will automatically execute jobs only once, not for every running instance. 

Hangfire also provides a Dashboard for monitoring the running background processes and their status.

**Note**: Hangfire creates it's **own tables** in the database on first run. See [background job](https://aspnetboilerplate.com/Pages/Documents/Background-Jobs-And-Workers) and [Hangfire integration](https://aspnetboilerplate.com/Pages/Documents/Hangfire-Integration) documents for more information.

For more information, please check Hangfire's [documentation](https://docs.hangfire.io/). 

