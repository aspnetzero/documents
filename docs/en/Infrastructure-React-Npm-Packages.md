# Package Management

ASP.NET Zero React UI uses [Yarn](https://classic.yarnpkg.com/) or [NPM](https://www.npmjs.com/) to manage front-end library dependencies (like React and Ant Design). You can easily add, update, or remove packages using the command line interface.

## Common Commands

| Action | Yarn | NPM |
|--------|------|-----|
| Install all dependencies | `yarn` | `npm install` |
| Add a new package | `yarn add package-name` | `npm install package-name` |
| Add a dev dependency | `yarn add package-name --dev` | `npm install package-name --save-dev` |
| Update packages | `yarn upgrade` | `npm update` |
| Remove a package | `yarn remove package-name` | `npm uninstall package-name` |

## Package.json

Dependencies are defined in [package.json](../package.json). After modifying dependencies, run `yarn` or `npm install` to update your `node_modules` folder.
