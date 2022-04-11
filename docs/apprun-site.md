# AppRun Site

The easiest way to get started is to use the [AppRun-Site](https://github.com/yysun/apprun-site) tool, a command-line tool for building modern web applications with [AppRun](https://github.com/yysun/apprun).

## Create Site

To create a new AppRun Site:

```sh
npm init apprun-app [my-app]
```

And select the `AppRun Site` template. An AppRun-Site project has the following structure:

```
/pages              <- pages of the website
  /index.html       <- index page
  /index.md         <- home page
  /main.tsx         <- start up code
  /about
    index.md        <- about page, markdown
  /contact
    contact.tsx     <- contact page, AppRun component
```

Then, you can use:

* _npm start_ or _npm run dev_ to start the dev server and watch your code changes.

The application will run at http://localhost:8080.


## Add Pages

You can add AppRun components, class or functional (tsx/jsx files), markdown, or html files to the `pages` directory.

### Example of a AppRun class component page:
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

### Example of a AppRun funtional component page:
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

### Example of a html page:
```html
<h2>Page</h2>
<div>This is an HTML page</div>
```

All the pages will be compiled to the ES modules into the `public` directory when you build the site.

Next, you will learn how to [build](apprun-site-build.md) your site.

