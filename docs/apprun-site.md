# AppRun Site

AppRun-Site let you focus on creating web pages using AppRun components, web components and even markdown files. AppRun-Site will take care of the rest:

* You can create web pages using AppRun components and markdown
* [Build](apprun-site-build.md) your pages to dynamic modules and load them on demand
* Render your pages to create a [static website](#static-website)
* [Serve](#serve) Single Page Applications (SPA) and Server-Side Rendering (SSR)
* [File-based routing](#file-based-routing)
* [API endpoints](#api-endpoints)
* [Server-side actions](#server-side-actions)


## Create AppRun Site

You can initialize a project using the `npm create apprun-app` command and select the `AppRun Site` template.

```sh
npm init apprun-app [my-app]
```

Alternatively, you can create a project using the `npx apprun-site init` command.

```sh
npx apprun-site init [my-app]
```

The `init` command provides a few more project templates to choose from:

* [AppRun Site Basic](https://github.com/apprunjs/apprun-site-template)
* [AppRun Site with Shadcn/ui](https://github.com/apprunjs/apprun-shadcn)
* [AppRun Site with Ant Design](https://github.com/apprunjs/apprun-antd-pro)


## AppRun Site Architecture

An AppRun-Site project has the following structure:

```
/pages              <- pages of the website
  /index.html       <- index file
  /index.tsx        <- home page
  /main.tsx         <- start up code (registers web component and renders the layout)
  /about
    /index.tsx      <- about page
  /contact
    /index.tsx      <- contact page
```

The pages are tsx/jsx files (AppRun components) or markdown files:


## Add Pages

You can add AppRun components, class or functional (tsx/jsx files), markdown, or html files to the `pages` directory.

### Example of an AppRun class component page:

```javascript
import { app, Component } from 'apprun';
export default class ContactComponent extends Component {
  state = '...';
  view = state => <div>
    <h2>{state}</h2>
    <p>This is a class Component</p>
  </div>;
}
```

### Example of an AppRun functional component page:

```javascript
import app from 'apprun';
export default () => <>
  <h2>...</h2>
  <p>This is a functional Component</p>
</>
```

### Example of a markdown page:

```markdown
# Hello Web Component
This is a markdown page with a web component to display a comic from XKCD
<ws-comic></ws-comic>
```


All the pages will be compiled into the ES modules in the `public` directory when you build the site.

Next, you will learn how to [build](apprun-site-build.md) your site.

