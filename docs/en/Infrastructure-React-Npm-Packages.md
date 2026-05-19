# Package Management

ASP.NET Zero React UI uses [pnpm](https://pnpm.io/) (preferred) or [NPM](https://www.npmjs.com/) to manage front-end library dependencies (like React and Ant Design). The pinned pnpm version is declared in `package.json` under the `packageManager` field, so [Corepack](https://nodejs.org/api/corepack.html) will activate the correct version automatically. You can easily add, update, or remove packages using the command line interface.

## Common Commands

| Action | pnpm | NPM |
|--------|------|-----|
| Install all dependencies | `pnpm install` | `npm install` |
| Add a new package | `pnpm add package-name` | `npm install package-name` |
| Add a dev dependency | `pnpm add -D package-name` | `npm install package-name --save-dev` |
| Update packages | `pnpm update` | `npm update` |
| Remove a package | `pnpm remove package-name` | `npm uninstall package-name` |

## Package.json

Dependencies are defined in [package.json](../package.json). After modifying dependencies, run `pnpm install` or `npm install` to update your `node_modules` folder.
