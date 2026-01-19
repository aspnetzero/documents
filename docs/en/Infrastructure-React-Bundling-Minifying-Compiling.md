# Asset Bundling and Minifying

ASP.NET Zero uses [Vite](https://vitejs.dev/) to build the React application. Vite automatically handles all bundling, minification, and optimization of your JavaScript, TypeScript, CSS, and other assets.

## Development

During development, run the following command to start the Vite development server:

```bash
yarn dev
# or
npm run dev
```

Vite provides fast Hot Module Replacement (HMR) and serves your assets without bundling them, which results in extremely fast development experience.

## Production Build

For production, run the following command:

```bash
yarn build
# or
npm run build
```

This command will:

* Compile TypeScript files
* Bundle all JavaScript/TypeScript modules into optimized chunks
* Minify JavaScript and CSS files
* Optimize and hash assets for caching
* Generate the production-ready output in the `dist` folder

## Preview Production Build

To preview the production build locally before deployment, you can use:

```bash
yarn preview
# or
npm run preview
```

This will serve the production build locally so you can verify everything works correctly.

## Vite Configuration

If you need to customize the bundling process, you can modify the **vite.config.ts** file. Vite provides extensive configuration options for:

* Custom build targets
* Asset handling and optimization
* Code splitting strategies
* CSS preprocessing
* And much more

For more information, refer to the [Vite documentation](https://vitejs.dev/config/).
