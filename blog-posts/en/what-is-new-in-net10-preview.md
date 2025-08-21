# What's New in .NET 10 Preview (LTS): ASP.NET Core, EF Core, and MAUI Features

Release date of .NET 10 is getting closer and Preview 4 version of .NET 10 is released. The .NET team continues to do an impressive job advancing the .NET ecosystem.

.NET 10 will be a long-term support (LTS) release. .NET developers like LTS releases and I think .NET 10 will be used more than .NET 9. 
This is evident when you compare the download counts of .NET NuGet packages on https://www.nuget.org/. 

Since our team is mostly focused on web development, we will take a deeper look at ASP.NET Core and EF Core features and improvements in .NET 10.
ASP.NET Zero also provides a MAUI project which allows you to authenticate via your backend API using Token Based Authentication. Because of that, we will also take a look at MAUI enhancements as well.

For the full list of "What's new in .NET 10", I suggest you to read [https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/overview](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/overview).

## What's new in ASP.NET Core for .NET 10

ASP.NET Core is a strong web application development framework. Let’s explore some of the exciting new features in the upcoming ASP.NET Core 10.0 release.

### Blazor script as static web asset

Currently the Blazor script is served from an embedded resource but with .NET 10, it will be served as a static web asset with automatic compression and fingerprinting.

### Response streaming

In previous versions of Blazor, to use response streaming for HttpClient, you need to enable it. With .NET 10, it is enabled by default. So, if you don't want this, you need to disable it.

In order to disable it, set the `DOTNET_WASM_ENABLE_STREAMING_RESPONSE` environment variable to `false` or `0`.

### Environment in standalone Blazor WebAssembly apps

`Properties/launchSettings.json` will not be used for environment. You need to use `<WasmApplicationEnvironmentName>` attribute in your project file (csproj) with .NET 10. 

For example, you can use a syntax like the one below for Development

```xml
<WasmApplicationEnvironmentName>Development</WasmApplicationEnvironmentName>
```

### Preloaded Blazor static assets

For Blazor Web Apps, you can now preloaded static assets. For standalone Blazor WebAssembly apps, you can schedule static assets for high priority downloading. 

### OpenAPI 3.1 support

With .NET 10, you can now generate OpenAPI 3.1 documents. The default OpenAPI document version is now 3.1 but you can change it as shown below;

```csharp
builder.Services.AddOpenApi(options =>
{
    options.OpenApiVersion = Microsoft.OpenApi.OpenApiSpecVersion.OpenApi3_1;
});
```

If you are generating OpenAPI documents on build time, you can use `OpenApiGenerateDocumentsOptions` item in your projetc file as shown below;

```xml
<PropertyGroup>
    <OpenApiGenerateDocumentsOptions>--openapi-version OpenApi3_1</OpenApiGenerateDocumentsOptions>
</PropertyGroup>
```

## What's new in Entity Framework Core for .NET 10

EF Core is really a great part of .NET ecosystem. I'm using it since its first release and it is getting stronger and robust in each release.

### Vector similarity search

Vector similarity search functionality was added in EF Core 9 as experimental and with EF Core 10, it is no longer experimental. As you may know, Vectors are important in AI world and this is a good sign that Microsoft cares about this in the development process.

### LeftJoin and RightJoin support

Left join is one of the mostly used operation when working with EF Core. In previous versions, if we wanted to make a left join, we need to use `SelectMany`, `GroupJoin` and `DefaultIfEmpty` operations. With EF Core 10, we can start using `LeftJoin` method as shown below;

```csharp
var query = context.Employees
    .LeftJoin(
        context.Departments,
        employee => employee.DepartmentID,
        department => department.ID,
        (employee, department) => new 
        { 
            employee.FirstName,
            employee.LastName,
            Department = department.Name ?? "[NONE]"
        });
```

Other than LeftJoin, `RightJoin` is also supported in EF Core 10.

There are also several query improvents. You can see the full list here [https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-10.0/whatsnew#other-query-improvements](https://learn.microsoft.com/en-us/ef/core/what-is-new/ef-core-10.0/whatsnew#other-query-improvements)

## What's new in .NET MAUI for .NET 10

### .NET Aspire integration

Maybe the most exciting feature in .NET 10 for MAUI is .NET Aspire integration. There is a new project template which creates a .NET Aspire service defaults project for .NET MAUI.

This integration brings cloud-native capabilities like telemetry, service discovery, and configuration management directly to your mobile apps.

### CollectionView and CarouselView

`CollectionView` and `CarouselView` added in .NET 9 as optional handlers on iOS and Mac Catalyst are now the default handlers.
`ListView` and `TableView` are also deprecated, so you should use `CollectionView` with .NET 10. If you're still using ListView or TableView, it's a good time to migrate to CollectionView to ensure future compatibility and take advantage of the newer rendering pipeline.

For the full list of important changes in .NET 10 for MAUI, you can check [https://learn.microsoft.com/en-us/dotnet/maui/whats-new/dotnet-10?view=net-maui-9.0](https://learn.microsoft.com/en-us/dotnet/maui/whats-new/dotnet-10?view=net-maui-9.0)

## Conclusion

.NET 10 brings a wide range of improvements across the stack—from better Blazor and API support in ASP.NET Core to stronger data capabilities in EF Core and modernized tooling for .NET MAUI. Since this will be an LTS release, it's a great time to start exploring the new features and preparing your applications for the upgrade.