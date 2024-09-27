
# AppRun-Site Client-Side Rendering

The AppRun-Site _build_ command creates pages as ES modules that can be loaded dynamically. The AppRun-Site _build_ command injects the client-side code into the _main.js_ file for routing and loading the pages. To conclude, AppRun Sites are Single Page Applications (SPA).

## App Startup

If you have a startup code, you can add it to the _main.tsx_ file. The AppRun-Site _build_ command injects calls to the default exported function of the _main.tsx_ file.

```
/pages              <- pages of the website
  /main.tsx         <- startup code
```

### Example of a markdown page:
```javascript
import app from 'apprun';
import Layout from '../components/layout'
import Comic from '../components/comic';

export default () => {
  app.webComponent('ws-comic', Comic);        // register web component
  app.render(document.body, <Layout />);      // render site layout
}

```

## No-Code Routing

You don't need to write any code to route an URL to a component. When users visit an URL, the client-side code will load, route, and render the page dynamically.

E.g., when the URL in the browser address bar becomes http://.../contact, it triggers the _/contact_ event. The _Contact component_ reacts to the _/contact_ and renders itself to the screen.

The event handler for the _/contact_ event is also injected. Therefore, there is no need to code for routing.

## Routing Parameters

However, if you want to pass parameters to the component through the URL, you can create your event handler. For example:
```javascript
import { app, Component } from 'apprun';
export default class extends Component {
  state = async () => { ... }
  view = state => <div>...</div>;
  update = {
    '/products': async (state, id) => {
      state = await Promise.resolve(state);
      return ({ ...state, id: parseInt(id) })
    }
  };
}
```
When the URL in the browser address bar becomes http://.../products/1, it triggers the _/products_ event and passes the _1_ as the parameter to your event handler.

## Pretty Links

AppRun-Site injects code by default to support pretty links (a.k.a. non-hash links), e.g., http://.../products/1. You don't need to write any code to support pretty links. However, you will need a webserver to serve the _index.html_ when the routes don't exist on the server.

> The AppRun-Site dev server provides such capability of serving _index.html_ when the routes don't exist.


## File-based Routing

AppRun-site also injects code to load the pages on demand as dynamic modules using thw following steps:

* load the index.html
* load the main.js (for the dynamic layout and the start up code)
* load the modules by path to render the pages:

```
/public
  /                 <- /index.js
  /about            <- /about/index.js
  /contact          <- /contact/index.js
```

> Note:
> * The main.js should create a layout and a div with the id `main-app` to render the pages.
> * The page modules should create a div with the id `[page]-app` for sub pages. E.g., /docs/index.js should create a div with the id `docs-app` for its sub pages if any.


The some file-based routing logic runs on the server side for the server-side rendering.

Next, you will see how to use the dev server and how it renders your pages on the [server side](apprun-site-ssr.md).
