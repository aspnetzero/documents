# Swagger UI

[Swagger UI](http://swagger.io/swagger-ui/) is **integrated** to ASP.NET Zero **by default**. You can browse **Swagger UI** from `https://localhost:44301/swagger/ui/` URL.

Notice that this is working in the server-side Host project. 

The below page is the default page of server side API application where you can see all available API endpoints:

<img src="images/swagger-ui-ng2-1.png" alt="Swagger UI" class="img-thumbnail" />

**.Web.Host** project contains a basic login page. If you login on this page, you can execute actions on Swagger UI which requires authentication.

In order to enable comments on Swagger UI, set ```Swagger:ShowSummaries``` to true in appsettings.json. After that, Swagger UI will show summaries written on your application services and controllers.