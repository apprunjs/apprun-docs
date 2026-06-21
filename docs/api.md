# API Reference

You only need three functions to get started — `app.start`, `app.run`, and `app.on`. This
page lists the full public API for when you need more.

```js
import app from 'apprun';
// or named imports:
import app, { Component, on, customElement, Fragment, trustedHTML,
  ROUTER_EVENT, ROUTER_404_EVENT } from 'apprun';
```

## app

The default export is the global `app` instance.

### Event system

| Method | Description |
|--------|-------------|
| `app.on(name, fn, options?)` | Subscribe to an event. |
| `app.once(name, fn, options?)` | Subscribe to an event for a single firing. |
| `app.off(name, fn)` | Unsubscribe a handler. |
| `app.run(name, ...args)` | Publish an event synchronously. Returns the number of handlers. |
| `app.runAsync(name, ...args)` | Publish an event and `await` all handler results. Returns `Promise<any[]>`. |
| `app.find(name)` | Return the subscribers registered for an event. |

`runAsync` replaces the older `query` helper for awaiting handler return values.

### Application & rendering

| Method | Description |
|--------|-------------|
| `app.start(element?, state?, view?, update?, options?)` | Create and start a component. Returns the `Component`. |
| `app.render(element, vdom, component?)` | Render a virtual DOM into an element. |
| `app.h(tag, props?, ...children)` / `app.createElement(...)` | Create virtual DOM nodes (the JSX factory). |
| `app.Fragment(props, ...children)` | JSX fragment factory. |
| `app.webComponent(name, ComponentClass, options?)` | Define a component as a custom element. |
| `app.use_render(render, mode?)` | Use a third-party renderer (`mode` 0 = React style, 1 = AppRun style). |
| `app.use_react(React, ReactDOM)` | Render views with React. |
| `app.trustedHTML(html)` | Mark caller-owned HTML as trusted for rendering. |
| ~~`app.safeHTML(html)`~~ | **Deprecated** — use `trustedHTML`. |
| `app.version` | The AppRun version string. |

### Routing

| Member | Description |
|--------|-------------|
| `app.route` | The router function. Overwrite to replace the default router. |
| `app.basePath` | Base path for sub-directory deployments. |
| `app.addComponents(element, routes)` | Map routes to components. |

See [Routing](routing.md) for details.

### app.start options (`AppStartOptions`)

```ts
{
  render?: boolean;       // render immediately (default true for app.start)
  history?: boolean | { prev?: string; next?: string }; // enable state history
  transition?: boolean;   // use the View Transitions API
  route?: string;         // route event that activates this component
  rendered?: (state) => void;          // lifecycle hook
  mounted?: (props, children, state) => state | void; // lifecycle hook
}
```

## Component

```js
import { Component } from 'apprun';

class MyComponent extends Component {
  state;      // initial state (value, function, or Promise)
  view;       // (state) => VDOM
  update;     // event handler map, [name, fn][] , or use @on
}
```

| Method | Description |
|--------|-------------|
| `new C(state?, view?, update?, options?)` | Construct a component. |
| `.mount(element?, options?)` | Mount without rendering until an event arrives. |
| `.start(element?, options?)` | Mount and render the initial state. |
| `.unmount()` | Remove the component. |
| `.setState(state, options?)` | Replace the state and re-render. |
| `.on(event, fn, options?)` | Subscribe to a local event. |
| `.run(event, ...args)` | Publish a local event (global if prefixed `#`, `/`, or `@`). |
| `.runAsync(event, ...args)` | Async publish; returns `Promise<any[]>`. |
| `.add_action(name, action, options?)` | Register an event handler programmatically. |

### Lifecycle hooks

| Hook | When |
|------|------|
| `mounted(props, children, state)` | After mount; may return a new state (e.g. merge props). |
| `rendered(state)` | After the view renders to the DOM. |
| `unload(state)` | When the mounted element is removed or reused. |

See [Component](component.md) for examples.

## Decorators & exports

| Export | Description |
|--------|-------------|
| `on(name?, options?)` | Method decorator that registers an event handler. |
| `customElement(name, options?)` | Class decorator that defines a custom element. |
| `Fragment(props, ...children)` | JSX fragment factory. |
| `trustedHTML(html)` | Mark trusted HTML for rendering. |
| ~~`safeHTML`~~, ~~`update`~~, ~~`event`~~ | **Deprecated** — use `trustedHTML` / `on`. |
| `ROUTER_EVENT` | Event published after every route. |
| `ROUTER_404_EVENT` | Event published for unmatched routes. |

## Key types

```ts
State<T>  = T | Promise<T> | (() => T) | (() => Promise<T>);
View<T>   = (state: T) => VDOM | void;
Action<T> = (state: T, ...args) => T | Promise<T> | void | AsyncGenerator<T> | Generator<T>;
Update<T> = ActionDef<T>[] | { [name: string]: Action<T> | [Action<T>, EventOptions?] };
```

`Action` may return a value, a `Promise`, or an (async) generator that **yields multiple
states** — AppRun renders each yielded state in order.
