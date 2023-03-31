# UI Tests

ASP.NET Zero provides an infrastructure for UI tests using [Playwright](https://playwright.dev/). You can check Playwright documentation to learn more about its features.

## Project Setup & Structure

You should have the following installed in your system:

- [Node.js](https://nodejs.org/en/) is the runtime. Please make sure you have v12+ (LTS recommended) in your system before you start.
- [Yarn v1.x](https://classic.yarnpkg.com/lang/en/)

After making sure these exist in your system, follow the steps below:

1. Open your terminal.
2. Navigate to the `ui-tests-playwright` folder of this project.
3. Run `yarn`.

There are two parts of the project that requires your close attention.

- The `playwright.config.ts` is the configuration file for Playwright. You can check [https://playwright.dev/docs/test-configuration](https://playwright.dev/docs/test-configuration) to learn its options.
- The `tests` folder includes the actual tests and utilities to create them with less effort. **This is where you will place your test suites.**

Another important file you need to know about is the `.env` file. This file contains website URL to test, a username and a password to login. 

After installing dependencies, the project will be ready-to-run.

### Running the tests

Before running the UI tests, be sure that your MVC app is  running with a brand new database.

When you run the UI tests, a screenshot will be created for each UI test and this screenshot will be compared to new ones for the following executions of the same UI test. 

Because of that, if you are running the UI tests for the first time, you must add `--update-snapshots` parameter to your test execution command.

```shell
npx playwright test --update-snapshots
```

For the following executions, you need to run same command without `--update-snapshots` parameter;

```shell
npx playwright test
```

This command runs all tests, but sometimes you may want to run a single test or a test group. For such cases, insert the relative file path to the end of the same command for that.

```shell
npx playwright test mvc/editions
```

If you want to run all specs for one project

```shell
npx playwright test mvc
```

In your own repository, you can store screenshots in your repository and this will allow you to easily run UI tests for your app.

## How to Create Tests

Please examine `tests/mvc/editions.spec.ts` to see an example spec.

There are two ways to create a test.

- Write directives to Playwright manually.
- Use the Playwright CLI to generate code.

ASP.NET Zero's default UI tests are executed in serial. So, tests are executed one by one and next test uses the state of previous test. We suggest using this approach when you are writing a UI test in your project as well.

Also, ASP.NET ZEro's default UI tests removes all data created during the UI tests. For example, this UI test creates a new Role with the name 'test' (Input fields are filled in a previous step which is not shown here). 

````typescript
/* Step 5 */
test('should save record when "Save" button is clicked', async () => {
    await rolesPage.saveForm();
    await rolesPage.waitForResponse();
    await rolesPage.replaceLastColoumnOfTable();

    await expect(page).toHaveScreenshot(ROLES_CRUD_NEW_SAVE);
});
````

And after that, this final step in Role UI test group removes the created record.

````typescript
/* Step 12 */
test('should delete record on click to "Yes" button', async () => {
    await rolesPage.openActionsDropdown(2);
    await rolesPage.waitForDropdownMenu();
    await rolesPage.triggerDropdownAction('Delete');
    await rolesPage.waitForConfirmationDialog();
    await rolesPage.confirmConfirmation();
    await rolesPage.waitForResponse();
    await rolesPage.replaceLastColoumnOfTable();

    await expect(page).toHaveScreenshot(ROLES_CRUD_DELETE_CONFIRM);
});
````

In this way, you can run your UI test during development.

**Tip:** If you want to see the browser when tests are running, set `headless: true` in **playwright.config.ts**.

**Tip:** If you use VSCode to write your tests you can use debugging.

