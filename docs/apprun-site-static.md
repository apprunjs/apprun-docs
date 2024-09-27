# AppRun-Site - Static Site


The AppRun-Site _build_ command has the option to render your pages into HTML files and create a static website.

```sh
npx apprun-site build --render
```

The _build_ command first scans and compiles your pages in the `pages` directory and then downloads them into the `public` directory by leveraging the dev server's server-side rendering.

```
/public             <- compiled site
  /_.html           <- copied from `pages/index.html` for SPA
  /index.html       <- *** server-side rendered `home` page ***
  /index.js         <- compiled
  /main.js          <- compiled
  /about
    /index.html     <- *** server-side rendered `about` page ***
    index.js        <- compiled
  /contact
    /index.html     <- *** server-side rendered `contact` page ***
    index.js        <- compiled

/pages              <- sorrce pages
  /index.html       <- index page
  /index.md         <- home page
  /main.tsx         <- start up code
  /about
    index.md        <- about page, markdown
  /contact
    contact.tsx     <- contact page, AppRun component`
```

With all the HTML pages created, you can deploy the static website.


In the next section, you will learn the [AppRun-Site command line](apprun-site-cli.md).
