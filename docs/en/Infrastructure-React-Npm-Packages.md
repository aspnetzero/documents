# NPM Packages

ASP.NET Zero React UI uses [NPM](https://www.npmjs.com/) to manage front-end library dependencies (like React and Ant Design). You can easily add, update, or remove packages using npm's command line interface.

## Common Commands

```bash
# Install all dependencies
npm install

# Add a new package
npm install package-name

# Add a dev dependency
npm install package-name --save-dev

# Update packages
npm update

# Remove a package
npm uninstall package-name
```

## Package.json

Dependencies are defined in [package.json](../package.json). After modifying dependencies, run `npm install` to update your `node_modules` folder.
