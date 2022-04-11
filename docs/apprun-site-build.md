# AppRun-Site Build

The AppRun-Site _build_ command compiles your page to ES Modules with [esbuild](https://esbuild.github.io/).

```sh
npx apprun-site build
```

The _build_ command scans your pages in the `pages` directory and compiles them into the `public` directory.


```
/public             <- compiled site
  /index.html       <- copied from `pages/index.html`
  /index.js         <- compiled from `pages/index.md`
  /main.js          <- compiled from `pages/main.tsx` and some bootstrap code
  /about
    index.js        <- compiled from `pages/about/index.md`
  /contact
    index.js        <- compiled from `pages/contact/index.tsx`

/pages              <- source pages
  /index.html       <- index page
  /index.md         <- home page
  /main.tsx         <- start up code
  /about
    index.md        <- about page, markdown
  /contact
    contact.tsx     <- contact page, AppRun component
```

The compiled js files are ES modules. They are compatible on the client-side and server-side.

Next, you will learn about [client-side rendering](apprun-site-csr.md).

