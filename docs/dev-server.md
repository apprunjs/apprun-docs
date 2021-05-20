---
title: AppRun Dev Server Supports ESM
published: true
description:
tags: ESM, JavaScript, TypeScript, Module
---

## Introduction

We use the [JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) (ESM) extensively while coding nowadays. But we still cannot deploy the module-based code yet. It is because the browsers don't know how to handle global modules.  E.g., When developing applications using [AppRun](https://github.com/yysun/apprun), we need a globe module of _apprun_.

```js
import app from 'apprun'
```

The browsers don't know how to import _apprun_. Therefore, We still need to use JavaScript bundlers such as webpack, rollup, or parcel to bundle the modules.

But at least now, we can use the modules to speed up the development process. Recently, the Snowpack team introduced the concept of [Unbundled Development](https://www.snowpack.dev/concepts/how-snowpack-works), which is to leverage modules for speeding up the development process.

In the past, I was thinking of building a tool to convert the global modules to the modules links on _unpkg_ after compilation.

> The [npm package CDN](https://unpkg.com), unpkg.com [supports delivering modules](https://github.com/mjackson/unpkg/issues/34) for along time. We can load _apprun_ as a module from _unpkg_.

```js
import app from 'https://unpkg.com/apprun?module'
```

Now, it seems that a development server is a different and better idea. So, I forked the live-server and made a development server for AppRun.

This post is to introduce the AppRun development server, called [_apprun-dev-server_](https://github.com/yysun/apprun-dev-server).

## apprun-dev-server

This is a static web server for developing JavaScript/TypeScript using ES modules following the concept of [Unbundled Development](https://www.snowpack.dev/concepts/how-snowpack-works).

* It serves the ES Modules from unpkg.com.
* Based on [live-server](https://www.npmjs.com/package/live-server), so it reloads the page automatically
* Also, it detects [AppRun](https://github.com/yysun/apprun) and can replace the module/Component while keeping the application _state_.

![](https://github.com/yysun/apprun-dev-server/raw/master/public/apprun-hmr.gif)

The best part of the apprun-dev-server is that it does NOT require any code in our components to handle the hot module replacement. Instead, it retains the component state; replaces the module; and then puts the state back. All done automatically.

If you want to refresh the state, you can reload the page in the browser by pressing F5 (on Windows) or Command+R (on Mac).

## How to Use

You export Component as the default module export.

```js
import { app, Component } from 'apprun';

export default class AboutComponent extends Component {
  state = 'About';
  view = state => <div>
    <h1>{state}</h1>
  </div>;
  update = {
    '#About': state => state,
  };
}
```

Then, you use the Component in the main file.

```js
import About from './About';

new About().start('my-app');
```

Then, you use a module-type script tag in HTML.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>AppRun SPA</title>
</head>
<body>
  <script type="module" src="/dist/main.js"></script>
</body>
</html>
```

Turn on the compiler, TypeScript, or Babel in watch mode. And then, start the apprun-dev-server using npx.

```
npx apprun-dev-server
```

Apprun-dev-server monitors the file changes. If the changed JavaScript files (*.js) file have global modules. Apprun-dev-server replaces the global module's references to _unpkg_. In the server console, if you see the file names that have some dots '......' in front, they are the files modified.

Apprun-dev-server injects JavaScript code snippets in the index.html just like live-server. Also, Apprun-dev-server adds logic to detect AppRun and replace AppRun components.

You can download an example app to give it a try.

```
npx degit yysun/apprun-esm-server my-app
```

## Configuration

Create a apprun-dev-server.config.js in your project:

```js
module.exports = {
  port: 8181, // Set the server port. Defaults to 8080.
  host: "0.0.0.0", // Set the address to bind to. Defaults to 0.0.0.0 or process.env.IP.
  root: "public", // Set root directory that's being served. Defaults to cwd.
  open: false, // When false, it won't load your browser by default.
  ignore: '', // comma-separated string for paths to ignore
  file: "index.html", // When set, serve this file (server root relative) for every 404 (useful for single-page applications)
  wait: 1000, // Waits for all changes, before reloading. Defaults to 0 sec.
  mount: [], // Mount a directory to a route.
  logLevel: 2, //
}
```
## Use with esbuild

The apprun-dev-server is installed when using AppRun CLI to create AppRun projects.

```sh
npx apprun --init --spa --esbuild
```

Give it a try and send pull requests.

https://github.com/yysun/apprun-dev-server


