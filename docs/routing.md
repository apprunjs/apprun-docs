# Routing

Routing in AppRun is event-driven. When the URL changes, the AppRun router publishes an
AppRun event using the URL as the event name. Components (or plain `app.on` handlers)
subscribe to those events. There is no separate routing configuration — routing is just
the event pub-sub system applied to the URL.

## How it works

The router listens to the browser and publishes events automatically:

* On `hashchange`, it routes `location.hash`.
* On `popstate`, it routes `location.pathname`.
* For path routing, it also intercepts same-origin `<a>` clicks, calls
  `history.pushState`, and routes the new path — so plain links just work, no extra code.

When the URL becomes `#/contact`, the router publishes the `#/contact` event. A component
that subscribes to `#/contact` reacts and renders itself. That's the whole mechanism.

```js
class Contact extends Component {
  view = () => <div>Contact</div>;
  update = { '#/contact': state => state };
}
```

## Hash routing vs. path routing

AppRun supports three URL styles, and **auto-detects** which mode to use:

| Style | Example event name | Notes |
|-------|-------------------|-------|
| Hash | `#contact` | Hash fragment, no slash |
| Hash-slash | `#/contact` | Hash fragment with slash (recommended for SPAs) |
| Path | `/contact` | Clean URLs via the History API |

If any `#` or `#/` route handler is registered, AppRun uses **hash routing** and listens to
`hashchange`. Otherwise it uses **path routing**, listens to `popstate`, and intercepts
link clicks.

* **Hash routing** works anywhere with no server configuration — the server only ever sees
  the page before the `#`. It is the safest choice for static hosting and the interactive
  examples in these docs.
* **Path routing** produces clean URLs (`/contact`) but requires the server to serve your
  `index.html` for every route (SPA fallback), so a deep-link reload like
  `https://site/contact` doesn't 404.

Pick one style and use it consistently across your routes.

## Route parameters

Routes can capture parameters with `:param` and a trailing `*` rest segment. Captured
values are passed to the event handler as arguments.

```js
// #/users/123  ->  id = "123"
app.on('#/users/:id', (state, id) => { /* ... */ });

// #/files/a/b/c  ->  rest = "a/b/c"
app.on('#/files/*', (state, rest) => { /* ... */ });
```

## Hierarchical routing

If there is no exact (or pattern) match, the router walks **up** the path and fires the
closest parent handler, passing the remaining segments as arguments.

```
For URL: /api/v1/users/123
Router tries: /api/v1/users/123 → /api/v1/users → /api/v1 → /api → 404
```

If a handler is registered for `/api`, it receives the leftover segments:

```js
// matches /api/v1/users/123  ->  args = 'v1', 'users', '123'
app.on('/api', (state, ...segments) => { /* ... */ });
```

The router stops before the root handlers (`/`, `#`, `#/`) and fires the 404 event instead,
so a missing route never accidentally activates the home page.

## Programmatic navigation

Publish the built-in `route` event to navigate from code:

```js
app.run('route', '#/contact');
```

For path routing this updates the History API and routes the new path.

## Sub-directory deployments (basePath)

If your app is served from a sub-directory, set `app.basePath`. The router strips it before
matching, and adds it back when navigating, so your route names stay relative.

```js
app.basePath = '/myapp';
// Navigation goes to /myapp/users/123
// Routing matches    /users/123
```

## Declarative routes with addComponents

`app.addComponents` maps routes directly to components (instances, classes, or functions
that return them), mounting each to a shared element.

```js
import app from 'apprun';
import Home from './Home';
import About from './About';

app.addComponents('#pages', {
  '#/':       Home,
  '#/about':  About,
  // lazy-loaded:
  '#/contact': () => import('./Contact').then(m => m.default),
});
```

## Unhandled routes (404)

When no handler matches a route, the router publishes `ROUTER_404_EVENT`, giving the app a
chance to degrade gracefully (for example, a 404 page).

```js
import app, { Component, ROUTER_404_EVENT } from 'apprun';

// Log unmatched routes.
app.on(ROUTER_404_EVENT, (url) => console.error('No route handler for', url));

// Or render a not-found page.
class NotFound extends Component {
  view = () => <h1>Page not found</h1>;
  update = {
    [ROUTER_404_EVENT]: state => state
  };
}
new NotFound().mount('#pages');
```

## The ROUTER_EVENT

After every routing attempt (matched or not), the router also publishes `ROUTER_EVENT` with
the route name and arguments. Subscribe to it for cross-cutting concerns such as analytics
or highlighting the active menu item.

```js
import app, { ROUTER_EVENT } from 'apprun';
app.on(ROUTER_EVENT, (url, ...args) => console.log('routed to', url, args));
```

## Replacing the default router

Routing is just `app.route`. To use a custom router, overwrite `app.route` and wire up the
browser events yourself:

```js
import app, { ROUTER_EVENT } from 'apprun';

function myRouter(url) {
  app.run(url);
  app.run(ROUTER_EVENT, url);
}

app.route = myRouter;

document.addEventListener('DOMContentLoaded', () => {
  window.onpopstate = () => myRouter(location.pathname);
  myRouter(location.pathname);
});
```
