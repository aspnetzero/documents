# Entity Framework Core

ASP.NET Zero template uses EntityFramework Core **code-first** and **migrations**. PhoneBook**DbContext** (YourProjectDbContext for your project) defines the DbContext class. **Migrations** folder contains EF migrations.

PhoneBook**RepositoryBase** class is the base class for your custom repositories. See [Entity Framework Integration](https://aspnetboilerplate.com/Pages/Documents/EntityFramework-Integration) documentation for more.

## Database Migrations

You can use Package Manager Console to add new migrations and update your database as you normally do. See EF Core's [documentation](https://docs.microsoft.com/en-us/ef/core/miscellaneous/cli/powershell) for details.