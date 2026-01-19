# Deployment

## About Deployment

ASP.NET Zero React UI is built using [Vite](https://vite.dev/) as the build tool and bundler. Vite provides fast development server and optimized production builds out of the box.

### Building for Production

To build the React application for production, run the following commands in the root directory of the React project:

1. Run `yarn` or `npm install` to install dependencies.
2. Run `yarn build` or `npm run build` to create a production build.

The build output will be placed in the `dist/` folder, ready for deployment to any static file server.

### Configuration

Before deploying, configure the production API endpoint by creating a `.env.production` file in the project root:

```env
VITE_API_URL=https://your-production-api-url
```

This URL should point to your ASP.NET Zero Host API application.

### Merged Solution (Optional)

ASP.NET Zero also provides a merged solution where the React client-side application is included in the server-side Host project. Both applications can be published together and hosted under the same website.

To publish the merged solution:

1. Run `yarn build` or `npm run build` in the React project to build the client application.
2. Run `dotnet publish -c Release` in the root directory of the `*.Host` project. See the [dotnet publish documentation](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-publish) for additional parameters.
3. When the publish is complete, ensure the React build output is copied to the `wwwroot` folder of the published Host application.

If you are using the merged solution, also configure settings in **appsettings.Production.json** file for the Host API.

## Production Optimizations

Vite automatically applies the following optimizations for production builds:

- **Code Splitting**: Automatically splits code into smaller chunks for better caching
- **Tree Shaking**: Removes unused code from the final bundle
- **Minification**: Minifies JavaScript and CSS files
- **Asset Optimization**: Optimizes and hashes static assets for cache busting

You can preview the production build locally by running:

```bash
yarn preview
# or
npm run preview
```

# Next

* [Publishing to Azure](Deployment-React-Publish-Azure)
* [Publishing to IIS](Deployment-React-Publish-IIS)
* [Publishing to Docker](Deployment-React-Docker)

