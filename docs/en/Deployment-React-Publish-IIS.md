# Step By Step Publish To IIS

## Host Application Publishing

Publishing ASP.NET Zero Host project is no different to any other ASP.NET Core Application.

### Publishing Web Site

- Right click `\*.Web.Host` project and click **Publish**.
- Click **Start** and select **Folder** tab and choose the URL that you want to publish.

<img src="images/iis-core-publish-select-folder-and-publish.png">

- Create a folder on the server where **IIS** is located. (for example: `C:\inetpub\wwwroot\mywebsite`).
- Copy published files to server. (from `*.Web.Host/bin/Release/net8.0/publish/` to `C:\inetpub\wwwroot\mywebsite`).
- Change `appsettings.production.json` configurations with your own settings.

### Create IIS Web Site

- Right click **Sites** and click **Add Website**.

<img src="images/iis-core-publish-add-website-to-iis.png">

- **mywebsite** application pool will be created automatically. Click **Application Pools**, double click **mywebsite** application pool and change its configuration.

<img src="images/iis-core-publish-configure-app-pool.png">

The project contains a `web.config` file, its contents may be changed during development, please check the` web.config` file after publishing, especially the `ASPNETCORE_ENVIRONMENT` setting.

Check [Host ASP.NET Core on Windows with IIS](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/index?view=aspnetcore-2.1) document for more detail.

## React Application Publishing

The React application is built using Vite and can be deployed as static files to IIS:

### Build the Application

1. Configure production settings by creating or updating `.env.production` in the project root:

```env
VITE_API_URL=https://your-host-api-url
```

2. Run `yarn build` or `npm run build`. This command outputs the production build to the `dist` folder.

3. Copy all files from the `dist` folder to your IIS website directory (for example: `C:\inetpub\wwwroot\reactwebsite`).

### Configure IIS for SPA Routing

**Important:** React uses client-side routing. If you refresh a page (F5), IIS will handle the request and will not find the requested path, returning an HTTP 404 error. You must configure IIS to redirect all requests to the `index.html` page.

**Prerequisites:** Install the [URL Rewrite module](https://www.iis.net/downloads/microsoft/url-rewrite) for IIS.

Create a `web.config` file in the website's root folder with the following content:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="React Routes" stopProcessing="true">
          <match url=".*" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="/index.html" />
        </rule>
      </rules>
    </rewrite>
    <staticContent>
      <remove fileExtension=".json" />
      <mimeMap fileExtension=".json" mimeType="application/json" />
    </staticContent>
  </system.webServer>
</configuration>
```

### Create IIS Web Site for React

1. Right click **Sites** and click **Add Website**
2. Set the physical path to your React build folder (e.g., `C:\inetpub\wwwroot\reactwebsite`)
3. Configure the site name and port
4. Ensure the application pool is set to **No Managed Code** since React is a static site

