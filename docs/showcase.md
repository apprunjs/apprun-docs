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


## AppRun App Showcase

This page shows a few applications built with AppRun.

### Conduit

* Project: https://github.com/gothinkster/apprun-realworld-example-app

<textarea>
--8<-- "real-world.html"
</textarea>
<apprun-play style="height:450px" hide_src="true" hide_button="true"></apprun-play>

### Hacker-News PWA

* Project: https://github.com/yysun/apprun-hn

<textarea>
--8<-- "hacker-news.html"
</textarea>
<apprun-play style="height:450px" hide_src="true" hide_button="true"></apprun-play>


### TodoMVC

* Author: https://github.com/samsondav
* Project: https://github.com/samsondav/todomvc-apprun

<textarea>
--8<-- "todo-mvc.html"
</textarea>
<apprun-play style="height:450px" hide_src="true" hide_button="true"></apprun-play>