# Vite Build Tool

ASP.NET Zero React UI uses [Vite](https://vitejs.dev/) for development and production builds. Vite provides fast development server with Hot Module Replacement (HMR) and optimized production builds.

## Development

To run the application in development mode, open a terminal and run:

```bash
yarn dev
# or
npm run dev
```

Once compiled and ready, you can open the application in your browser at <http://localhost:4200>.

## Production Build

To create a production build:

```bash
yarn build
# or
npm run build
```

This creates an optimized build in the `dist/` folder.

## Preview Production Build

To preview the production build locally:

```bash
yarn preview
# or
npm run preview
```

## Configuration

Vite configuration is in [vite.config.ts](../vite.config.ts). See the [Vite documentation](https://vitejs.dev/) for more configuration options.

