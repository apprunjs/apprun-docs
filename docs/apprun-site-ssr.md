# AppRun-Site Server-Side Rendering

The AppRun-Site _serv_ command starts a dev server at http://localhost:8080.

```sh
npx apprun-site serv
```

The AppRun-Site dev server serves _index.html_ when the routes don't exist to support Single Page Apps. It also has the capability of server-side rendering to support pretty links and static site creation.

## ES Module

Because the pages are compiled into ES Modules, they can be loaded dynamically. Also, thanks to the [AppRun architecture](architecture.md), the dev server can render all the pages on the server the same way in the browser.

E.g., when users visit http://.../contact, the dev server finds an ES module from `/contact/index.js`. The dev server loads the module dynamically and renders it to [jsdom](https://github.com/jsdom/jsdom). Then, it sends the HTML from _jsdom_ back to the browser.


The **exact** same code, /contact/index.js, can be run on the client-side and the server-side. No special treatment is needed.

## Dynamic Routing

The dev server supports dynamic routing. It searches the URL and finds the corresponding ES module. E.g., when users visit http://.../products/1, the dev server will first try to find the ES module from `/products/1/index.js`. Of course, /products/1/index.js does not exist, so it will try to find `/products/index.js.` If `/products/index.js` exists, it will load the module dynamically and sends _1_ as the parameter for rendering.

The result is that pretty links are supported on the server side.


## Server-Side Rendering

The server can render the pages on the server-side. It uses the same steps as the client-side rendering:

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
> In order to support server-side rendering, the code should be carefule about using browser-specific APIs. You should check if the code runs on the server-side is compatible.


## Server App

The `build` command generated `server.js` can serve API endpoints and server-side actions. The server code is in the `app` folder. You can add an `app` folder with the following structure:

```
/app                <- app folder (backend code)
  /_                <- action folder
  /api              <- api folder
/pages              <- pages of the website
```

The `build` command will compile the server code to allow the API endpoints and server-side actions.

If you only want to build the server code, you can use the `--server-only` option:

```sh
npx apprun-site build --server-only
```

### API Endpoints

The API endpoints are served at the path `/api/[endpoint]`. For example, the `api/hello.js` file will be served at `/api/hello`.

```js
// api/hello.js
export default (req, res) => {
  res.json({ hello: 'world' });
};
```

### Server-side Actions

You can add server-side actions in the `_` folder. The server-side actions are served at the path `/_/[action]`. For example, the `_/comic.js` file will be served at `/_/comic`.

Use the `//#if server`, `//#else`, and `//#endif` comments to separate the server-side code from the client-side code. The compiler will strip out the server-side code from the client-side code and vice versa.

```js
// _/comic.js
import action from 'apprun-site/action.js';
export default async (data) => {
  //#if server
  const num = data?.num || Math.floor(Math.random() * 2990) + 1;
  const response = await fetch(`https://xkcd.com/${num}/info.0.json`);
  return response.json();
  //#else
  return action('comic', data);
  //#endif
}
```

The `action` function will POST to the `comic` action on the client side.

You can refer to the server-side action in the client-side code:

```tsx
// components/comic.tsx
import { app, Component } from 'apprun';
import comic from '../../app/_/comic';
export default class Comic extends Component {
  state = comic();
  view = ({ img, alt }) => img ? <img src={img} alt={alt} /> : `Loading...`;
}
```

The benefit of referring to the server-side code is that you can get type checking, code completion and goto-definition in your IDE.


Finally, you may not have a node server or use a different web server in production. In this case, you can pre-render your site into a static site. Proceed to the next section to learn how to create a [static site](apprun-site-static.md).









