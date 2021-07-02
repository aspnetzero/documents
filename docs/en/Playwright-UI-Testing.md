# Aspnet Zero UI Tests

AspNet Zero provides an infrastructure for UI tests. It employs browser automation, screenshots, and image comparison to do so.

## Project Setup & Structure

You should have the following installed in your system:

- [Node.js](https://nodejs.org/en/) is the runtime. Please make sure you have v12+ (LTS recommended) in your system before you start.
- [npm] is the package manager and will help you install your dependencies and run scripts. It will be available when you install Node.js in your system.

After making sure these exist in your system, follow the steps below:

1. Open your terminal.
2. Navigate to the root folder of this project (if not already there).
3. Run `npm install`.

The following dependencies will be installed:

- [TypeScript](https://github.com/microsoft/TypeScript) is the language we use in this project. All utilities, spec files, and extensions to Jest are in TypeScript.
- [ES Lint](https://github.com/eslint/eslint) is the JavaScript/TypeScript linter. It finds and fixes problems in your code and thus ensures code quality and consistency.
- [Prettier](https://github.com/prettier/prettier) formats your code. The current configuration in this project makes Prettier play well with ES Lint.
- [dotenv](https://github.com/motdotla/dotenv) loads environment variables from the `.env` file.
- [Jest](https://github.com/facebook/jest) is the testing framework. Jest is also a test runner, so you do not need a separate one. It also is highly extensible. This project benefits from the extensibility of Jest a lot.
- [Playwright](https://github.com/microsoft/playwright) provides browser automation and screenshots. Playwright has a CLI which lets us generate the test code without writing (almost) any.
- [Pixelmatch](https://github.com/mapbox/pixelmatch) compares screenshots. It takes two images and returns an image and a numeric value representing the difference (a.k.a. diff).
- [pngjs](https://github.com/lukeapage/pngjs) is a PNG utility. It reads the PNG screenshots and materializes (writes) the diff as a PNG file.
- [EJS](https://github.com/mde/ejs) is a simple JavaScript template engine. It renders the issue template based on given variables.
- [fs-extra](https://github.com/jprichardson/node-fs-extra) provides promise-based file system methods. It reads, writes, and deletes files with great ease.
- [Lodash](https://github.com/lodash/lodash) is a utility library for JavaScript. It provides methods to help you work with strings, arrays, objects, and more.
- [Simple Git](https://github.com/steveukx/git-js) is a Node.js wrapper around git commands. We use it to create local branches, commit changes, and push commits to the remote.
- [Octokit](https://github.com/octokit/rest.js) is the SDK of GitHub. We create issues and pull requests with it.

There are two parts of the project that requires your close attention.

- The `jest.config.js` and the files under the `jest` folder establish the base for testing. You are not expected to make any changes to these files.
- The `tests` folder includes the actual tests and utilities to create them with less effort. **This is where you will place your test suites.**

Another important file you need to know about is the `.env` file. This file contains project configuration. It does not exist in the repo for security reasons. Please find the `.env.example` file and create a local `.env` file based on the descriptions. You will need a GitHub personal access token. Please refer to [the GitHub documentation](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) for details.

After installing dependencies and creating a valid `.env` file, the project will be ready-to-run.

## Running Tests in Development Mode

Before you start, make sure your frontend and backend projects are running.

The following steps take place upon running `npm test` in terminal:

1. The `jest` folder gets compiled.
2. Jest starts based on the configurations in `jest.config.js`.
3. All `.spec.ts` files in the `tests` folder get tested.
4. Any changes to screenshots are saved.

However, this command runs all tests, and you will most likely want to run a single test most of the time. Insert the relative file path to the end of the same command for that.

```shell
npm test mvc/editions
```

If you want to run all specs for one project

```shell
npm test mvc
```

## Running Tests in CI Mode

Before you start, make sure your frontend and backend projects are running.

The following steps take place upon running `npm run test:ci` in terminal:

1. Every step in the "Development Mode" happens.
2. Changes on each project are committed to separate branches.
3. Generated branches are pushed to the remote.
4. A single issue (per project with change) is created on the original project repo. The `/jest/diff_report.md` file is the issue template.
5. A pull request is created for each branch. The PR refers to the issue on the original project repo.

Please note that this command creates actual commits, branches, issues, and pull requests. Therefore, you should run the "Development Mode" unless you want these to happen.

If you want the tests to run only for a single file, insert the relative file path to the end of the same command.

```shell
npm run test:ci mvc/editions
```

Using base project folder instead will run all specs for that particular project.

```shell
npm run test:ci mvc
```

## How to Create Tests

Please examine `tests/mvc/editions.spec.ts` to see an example spec.

There are two ways to create a test.

- Write directives to Playwright manually.
- Use the Playwright CLI to generate code.

The first option requires you to know about the Playwright API. The second one, on the other hand, is quite easy. Please refer to [this section](https://github.com/microsoft/playwright-cli#generate-code) for details.

Although Playwright CLI helps you create tests, you will notice that it sometimes produces element selectors that are too tied to implementation. Please refer to [the documentation for selectors](https://github.com/microsoft/playwright/blob/master/docs/selectors.md#element-selectors) to see different approaches you can take. Also, the example spec demonstrates how to use imported functions for reusable selectors.

**Tip:** If you want to run tests in debug mode, uncomment `DEBUG=pw:api` line in your `.env` file.

**Tip:** If you want to see the browser when tests are running, uncomment `HEADFUL=` line in your `.env` file.

**Tip:** If you use VSCode to write your tests you can use debugging.