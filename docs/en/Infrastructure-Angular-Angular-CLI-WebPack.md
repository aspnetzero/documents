# Angular CLI

ASP.NET Zero uses [Angular CLI](https://angular.dev/tools/cli) for the development and deployment of the Angular application. The project is already configured for Angular CLI. The application is built with the `@angular/build:application` builder, which uses [esbuild](https://esbuild.github.io/) and [Vite](https://vite.dev/) instead of WebPack.

## Running the Application

To run the application, open a command line in the `angular` folder and run the following command:

```bash
pnpm start
```

Once it's compiled and ready, you can go to <http://localhost:4200> to open the application in your browser.

### Hot Module Replacement

`pnpm start` runs the development server with Hot Module Replacement (HMR) turned off (`ng serve --no-hmr`), so the page is reloaded after every change. To use the HMR support of Angular CLI, run the following command instead:

```bash
pnpm run hmr
```

This command updates the dynamic bundles (`gulp buildDev`) and then runs `ng serve` with HMR turned on. With HMR, changes to the templates and styles of components are applied to the running application without reloading the page. Other changes still reload the page.

HMR is built into Angular CLI, so it does not need an extra package or build configuration.

> Earlier versions of ASP.NET Zero had an `hmr` build configuration in `angular.json`, an `environment.hmr.ts` file and the `@angularclass/hmr` packages. They were not used by the Angular CLI build anymore and are removed. If your project still has them, you can delete them.

## Running Unit Tests

The unit tests of the Angular application run with [Vitest](https://vitest.dev/), using the `@angular/build:unit-test` builder of Angular CLI. The tests run in Node.js with [jsdom](https://github.com/jsdom/jsdom) as the DOM, so no browser is needed. To run the tests, run the following command in the `angular` folder:

```bash
pnpm test
```

All files that end with `.spec.ts` or `.test.ts` are tests. When you run the command in a terminal, Vitest keeps watching the files and runs the tests again after every change. You can pass the following options to the command:

| Command | Description |
|---------|-------------|
| `pnpm test --no-watch` | Runs the tests once and exits. Use it on build servers. |
| `pnpm test --coverage` | Also creates a code coverage report (an HTML report and an `lcov.info` file) in the `coverage/abp-zero-template` folder. |
| `pnpm test --include src/app/admin/roles` | Only runs the tests in a folder or file. |

See the [testing guide](https://angular.dev/guide/testing) of Angular for the other options and for writing tests.

### Test configuration

| File | Description |
|------|-------------|
| `angular.json` | The `test` target uses the `@angular/build:unit-test` builder with the `vitest` runner. It builds the tests with the `testing` build configuration, which does not include the global `styles`, `scripts` and `assets` of the application. |
| `src/test-setup.ts` | Runs before the tests. It loads the global `abp.js` script, so the `abp` APIs (such as `abp.setting` and `abp.localization`) can be used in tests. |
| `src/tsconfig.spec.json` | The TypeScript configuration of the tests. It includes the global functions of Vitest, so `describe`, `it`, `expect` and `vi` can be used without importing them. |

The global `scripts` of `angular.json` are not loaded in tests. If the code that you test uses another global script, add its path to the `globalScripts` array in `src/test-setup.ts`.

### Migrating tests from Jasmine

Earlier versions of ASP.NET Zero ran the tests with Karma and Jasmine. `karma.conf.cjs`, `src/test.ts` and the Karma and Jasmine packages are removed. If you have written your own tests, change the Jasmine APIs that they use to Vitest. `describe`, `it`, `beforeEach`, `afterEach`, `expect` and `TestBed` work in the same way.

| Jasmine | Vitest |
|---------|--------|
| `spyOn(obj, 'method')` | `vi.spyOn(obj, 'method').mockImplementation(() => {})` |
| `spyOn(obj, 'method').and.callThrough()` | `vi.spyOn(obj, 'method')` |
| `spyOn(obj, 'method').and.returnValue(value)` | `vi.spyOn(obj, 'method').mockReturnValue(value)` |
| `jasmine.createSpy('name')` | `vi.fn().mockName('name')` |
| `spy.and.resolveTo(value)` | `spy.mockResolvedValue(value)` |
| `spy.and.rejectWith(error)` | `spy.mockRejectedValue(error)` |
| `jasmine.Spy` | `Mock` (`import type { Mock } from 'vitest'`) |
| `jasmine.clock().install()` | `vi.useFakeTimers()` |
| `jasmine.clock().tick(ms)` | `vi.advanceTimersByTime(ms)` |
| `jasmine.clock().uninstall()` | `vi.useRealTimers()` |
| `jasmine.objectContaining(value)` | `expect.objectContaining(value)` |
| `expect(value).toBeTrue()` | `expect(value).toBe(true)` |
| `expect(value).toBeFalse()` | `expect(value).toBe(false)` |
| `expect(value).withContext('message')` | `expect(value, 'message')` |

Note these differences:

- Jasmine's `spyOn` does not call the original method. `vi.spyOn` calls it unless you add `mockImplementation` or `mockReturnValue`.
- Jasmine removes spies after each test. In Vitest, call `vi.restoreAllMocks()` in `afterEach` to restore the methods that `vi.spyOn` replaced.

Also, `pnpm test` no longer runs `gulp buildDev` before the tests, because the tests do not use the dynamic bundles.
