# Dev Tools

## AppRun CLI

AppRun includes a command-line tool (CLI) for creating a TypeScript and webpack configured project.

You can initialize a SPA project.

```sh
npx apprun --init --spa
```

To initialize a project that targets ES5, use the AppRun CLI with the --es5 flag:

```sh
npx apprun --init --spa --es5
```


You can initialize a SPA project that uses [esbuild](https://esbuild.github.io/).

```sh
npx apprun --init --spa --esbuild
```

## AppRun Dev Tools

To use the AppRun dev-tools, include the dev-tools script.

```JavaScript
<script src="https://unpkg.com/apprun/dist/apprun-dev-tools.js"></script>
```

AppRun Dev Tools allows you to watch events and states in the console.

<video width="100%" controls loop>
  <source src="imgs/apprun-debug-log.mp4" type="video/mp4">
</video>



To learn the AppRun CLI in Console, please read [the post](https://dev.to/yysun/make-cli-run-in-the-console-42ho).

Also, check out the [unit testing](11-unit-testing) for how to use the CLI in Console to generate unit tests.

## Redux Extensions

AppRun supports the Redux DevTools Extension. To use the DevTools, install the [Redux DevTools Extension](https://github.com/zalmoxisus/redux-devtools-extension). Then you can monitor the events and states in the dev tools.

![app-dev-tools](imgs/apprun-dev-tools.gif)

## VS Code Extension

AppRun has a code snippet extension for VS Code that you can install from the extension marketplace. It inserts the AppRun code template for application, component, and event handling.

![app-dev-tools](imgs/apprun-vscode-extension.png)

All the above dev tools let you explore and debug the internals of your applications.

Next, you will learn the [AppRun architectural concept](04-architecture).