# Application Services as MVC API Controllers

ASP.NET Zero project highly use AJAX to provide a better user experience. UI calls **application service methods** via AJAX. So, it's needed to create MVC API controllers as adapters (A Client calls MVC API
Controller action via AJAX, then it calls application service method). ABP framework automatically creates MVC API Controllers for all application services. So, no need to manually create MVC API Controllers
for application services.See related [documentation](https://aspnetboilerplate.com/Pages/Documents/AspNet-Core#application-services-as-controllers) for more. While ABP dynamically create Web API Controllers, we can also create regular MVC API Controllers as we always do.