# Getting Started

This guide describes how to get up and running with AppRun in minutes.

You can choose one of the following options:

1. [Use AppRun in the browser](tutorial.md) without any installation
2. [Use the _npm create apprun-app_](create-apprun-app.md) to create a new AppRun project
3. [Create an AppRun Site](apprun-site.md) for SSR and static site generation


## Installation

AppRun is distributed on npm. To get it, run:

```sh
npm install apprun
```

You can use AppRun directly from the unpkg.com CDN:

```js
<script src="https://unpkg.com/apprun/dist/apprun-html.js"></script>
<script>
  const view = state => `<div>${state}</div>`;
  app.start(document.body, 'hello AppRun', view);
</script>
```

Or, use the ESM version:
```js
<script type="module">
  import { app } from 'https://unpkg.com/apprun/dist/apprun-html.esm.js';
  const view = state => `<div>${state}</div>`;
  app.start(document.body, 'hello AppRun ESM', view);
</script>
```





