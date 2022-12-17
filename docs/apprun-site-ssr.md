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


Finally, although you have a dev server for development, you may not have a node server or use a different web server in production. In this case, you can pre-render your site into a static site. Proceed to the next section to learn how to create a [static site](apprun-site-static.md).









