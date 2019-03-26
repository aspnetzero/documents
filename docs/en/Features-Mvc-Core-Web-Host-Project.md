# Web.Host Project

ASP.NET Zero solution contains an extra project, **Web.Host**, which just exposes all application functionality as **remote API**. Thus, you can consume your application as API from any device. ***.Web.Mvc** project also does it, provides API for all application functionality. The difference is that ***.Web.Mvc** project has also MVC Controllers, Views, scripts and so on. If you just want to deploy API without UI, you can use Web.Host project. Otherwise, you can even delete it. We are using Web.Host project to provide server side API to the [Angular client](Index-Angular).

A few notes on **Web.Host** project:

- It has only **token based (JWT) authentication** (plus social login possibility). No form based authentication (because there is no UI).
- It does **not** implement **CSRF** protection since it's not a security concern in token based auth.
- It enables **CORS**. So, cross origin requests are allowed. It only allows to **http://localhost:4200** by default (see Startup class for the configuration).
- Configured and enabled **Swagger UI** by default.

## Next

- [Migrator Console Application](Migrator-Console-Application)

