# Use AppRun in the browser

## Which bundle to use

AppRun ships several builds. Choose by how you write your views and whether you want ES
modules:

| Entry | Globals exposed | Template style | Use when |
|-------|-----------------|----------------|----------|
| `dist/apprun-html.js` | `app`, `html`, `svg`, `run`, `Component` | `html` template literals | Quick start in a `<script>` tag, no build step. |
| `dist/apprun-html.esm.js` | (ESM imports) | `html` template literals | `<script type="module">`, no build step. |
| `dist/apprun.esm.js` | (ESM imports) | Bring your own renderer | ESM + a third-party renderer via `app.use_render`. |
| `npm install apprun` | (ESM imports) | JSX (with a compiler) | Projects using Vite/webpack/esbuild and JSX. |

The `html` and `svg` functions are lit-html template tags. The `run` function is the
lit-html equivalent of the `$on` directive. JSX requires a compiler, so it is not available
in the script-tag builds.

## Get AppRun from CDN

You can use AppRun directly from the unpkg.com CDN:

```js
<script src="https://unpkg.com/apprun/dist/apprun-html.js"></script>
<script>
  const view = state => `<div>${state}</div>`;
  app.start(document.body, 'hello AppRun', view);
</script>
```

## Use the ESM version

```js
<script type="module">
  import { app } from 'https://unpkg.com/apprun/dist/apprun-html.esm.js';
  const view = state => `<div>${state}</div>`;
  app.start(document.body, 'hello AppRun ESM', view);
</script>
```

## Use Other Libraries


### lit-html

```js
<html>
<body>
  <script type="module">
    import { app } from 'https://esm.run/apprun';
    import { html, render } from 'https://esm.run/lit-html';

    // use lit-html
    app.use_render(render);
    class Counter extends Component {
      state = 0;
      view = state => {
        return html`<div>
          <h1>${state}</h1>
          <button @click=${()=>this.run("-1")}>-1</button>
          <button @click=${()=>this.run("+1")}>+1</button>
        </div>`;
      };
      update = {
        '+1': state => state + 1,
        '-1': state => state - 1
      };
    }
    new Counter().start(document.body);
  </script>
</body>
</html>
```
<apprun-play style="font-size:.5rem" demo-width="30%"></apprun-play>


### htm

```js
<html>
<body>
  <script type="module">
    import { app } from 'https://unpkg.com/apprun/dist/apprun.esm.js';
    import { html, render } from 'https://unpkg.com/htm/preact/standalone.module.js';
    app.use_render(render);

    const add = (state, delta) => state + delta;
    const view = state => html`<div>
      <h1>${state}</h1>
      <button onClick=${() => app.run('-1')}>-1</button>
      <button onClick=${() => app.run('+1')}>+1</button>
    </div>
    `;
    const update = {
      '+1': state => state + 1,
      '-1': state => state - 1
    };
    app.start(document.body, 0, view, update);
  </script>
</body>
</html>
```
<apprun-play style="font-size:.5rem" demo-width="30%"></apprun-play>


## Installation

AppRun is distributed on npm. To get it, run:

```sh
npm install apprun
```



