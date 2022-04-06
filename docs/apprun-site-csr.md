
# AppRun-Site Client-Side Rendering

The AppRun-Site _build_ command creates a Single-Page Application (SPA), including client-side rendering and routing. During the _build_ process, it injects the client-side code into the _main.js_ file.

## No-Code Routing

You don't need to write any code to route an URL to a component. When users visit an URL, the client-side code will load, route, and render the page dynamically.

E.g., when the URL in the browser address bar becomes http://.../contact, it triggers the _/contact_ event. The _Contact component_ reacts to the _/contact_ and renders itself to the screen.

The event handler for the _/contact_ event is also injected. Therefore, there is no need to code for routing.

## Routing Parameters

However, if you want to pass parameters to the component through the URL, you can create your event handler.

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

To conclude, AppRun-Site builds Single Page Application (SPA). Next, you will see how to use the dev server and how does it render your pages on the [server side](apprun-site-ssr.md).
