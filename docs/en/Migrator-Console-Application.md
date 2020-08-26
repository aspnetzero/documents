# Migrator Console Application

ASP.NET Zero includes a tool, **Migrator.exe**, to easily migrate your databases. You can run this application to create/migrate host and tenant databases.

<img src="images/database-migrator.png" alt="Database Migrator" class="img-thumbnail" width="659" height="230" />

This application gets host connection string from it's **own appsettings.json file**. It will be same as in the appsettings.json in .Web project at the beginning. Be sure that the connection string in config file is the database you want. After getting **host** **connection string**, it first creates the host database or apply
migrations if it does already exists. Then it gets connection strings of tenant databases and runs migrations for those databases. It skips a tenant if it has not a dedicated database or it's database is already
migrated for another tenant (for shared databases between multiple tenants).

You can use this tool on development or on product environment to migrate databases on deployment, instead of EntityFramework's own Migrate.exe (which requires some configuration and can only work for
single database in one run).

## Next

- [Public Project](Public-Website)
