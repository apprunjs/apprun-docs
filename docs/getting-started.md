# Getting Started

This guide describes how to get up and running with AppRun in minutes.

> **AppRun Concepts**
>
> If you are looking for an introductory overview of AppRun technology, it is recommended to visit the concepts section.


## Installation

AppRun is distributed on npm.
```sh
npm install apprun
```

You can also load AppRun directly from the unpkg.com CDN:

```js
<script src="https://unpkg.com/apprun"></script>
```

Or when you need HTML or lit-HTML:

```js
<script src="https://unpkg.com/apprun/dist/apprun-html.js"></script>
```

## Use in Browser

### Use CDN

```html
<html lang="en">
<head>
  <title>AppRun App</title>
  <script src="https://unpkg.com/apprun/dist/apprun-html.js"></script>
</head>
<body>
<script>
  const view = state => `<div>${state}</div>`;
  app.start(document.body, 'hello world', view);
</script>
</body>
</html>
```
<apprun-play></apprun-play>

### Use ES Module in Browser

```html
<html lang="en">
<head>
  <title>AppRun App</title>
</head>
<body>
<script type="module">
  import { app } from 'https://unpkg.com/apprun/dist/apprun-html.esm.js';
  const view = state => `<div>${state}</div>`;
  app.start(document.body, 'hello ESM', view);
</script>
</body>
</html>
```
<apprun-play></apprun-play>

### Use lit-html

[lit-html](https://lit-html.polymer-project.org/) is an efficient, expressive, extensible HTML templating library for JavaScript from the Polumer project. AppRun has included lit-html.
```html
<html lang="en">
<head>
  <title>AppRun App</title>
</head>
<body>
<script type="module">
  import { app, html } from 'https://unpkg.com/apprun/dist/apprun-html.esm.js';
  const view = state => html`<div>${state}</div>`;
  app.start(document.body, 'hello lit-html', view);
</script>
</body>
</html>
```
<apprun-play></apprun-play>

### Use µhtml

[µhtml](https://github.com/WebReflection/uhtml) (micro html) is a ~2.5K _lighterhtml_ subset to build declarative and reactive UI via template literals tags. AppRun allows you to use different rendering technology like µhtml.

```html
<html lang="en">
<head>
  <title>AppRun App</title>
</head>
<body>
<script type="module">
  import app from 'https://unpkg.com/apprun?module';
  import { render, html } from 'https://unpkg.com/uhtml?module';
  app.render = render;
  const view = state => html`<div>${state}</div>`;
  app.start(document.body, 'hello uhtml', view);
</script>
</body>
</html>
```
<apprun-play></apprun-play>

### Use JSX

[JSX](https://reactjs.org/docs/introducing-jsx.html) is a syntax exntension to JavaScript. It makes JavaScript functions look like HTML.

You can use JSX in the browser or compile JSX ahead of time. See AppRun CLI below.

```html
<html lang="en">
<head>
  <title>AppRun App</title>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script src="https://unpkg.com/apprun"></script>
</head>
<body>
<script type="text/babel" data-presets="es2017, react">
  const view = state => <div>{state}</div>;
  app.start(document.body, 'hello JSX', view);
</script>
</body>
</html>
```
<apprun-play></apprun-play>

## Compile/Transpile and Bundle

### TypeScript and webpack

AppRun includes a command-line tool (CLI) for creating a TypeScript and webpack configured project.

```sh
npx apprun --init
```

### esbuild

You can initialize a project that uses [esbuild](https://esbuild.github.io/).

```sh
npx apprun --init --esbuild
```

After the command finishes execution, you can start the application and then navigate to https://localhost:8080 in a browser.

```sh
npm start
```

AppRun projects use the following convention.

* Use _npm start_ to start the dev server
* Use _npm test_ to run unit tests
* Use _npm run build_ to build for production


## Starter Template

Optionally, if not using the CLI you can directly scaffold AppRun project from the AppRun starter templates.
```sh
npx degit apprunjs/apprun-esbuild my-app
npx degit apprunjs/apprun-rollup my-app
npx degit apprunjs/apprun-rollup-lit-html my-app
npx degit apprunjs/apprun-webpack my-app
npx degit apprunjs/apprun-parcel my-app
npx degit apprunjs/apprun-web-components my-app
npx degit apprunjs/apprun-bootstrap my-app
npx degit apprunjs/apprun-coreui my-app
npx degit apprunjs/apprun-pwa my-app
npx degit apprunjs/apprun-pwa-workbox my-app
npx degit yysun/apprun-d3 my-app
npx degit yysun/apprun-electron my-app
npx degit yysun/apprun-electron-forge my-app
npx degit yysun/apprun-websockets my-app
```

AppRun is so flexible that you can choose your favorite ways of using it.


Next, you will see the quick-start tutorial on creating AppRun apps.