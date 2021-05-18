# HTML

AppRun supports creating the HTML string from the _view_ function. Sometimes it may be easy for quick prototyping.

```javascript
const view = state => `<div>
  <h1>${state}</h1>
  <button onclick="app.run('-1')">-1</button>
  <button onclick="app.run('+1')">+1</button>
</div>`;
```
Although HTML string is easy to understand and helpful in trying out ideas, it takes time to parse it into virtual DOM at run time, which may cause performance issues.

You can use alternative rendering technologies such as:

* [lit-html](https://github.com/Polymer/lit-html)
* [µhtml](https://github.com/WebReflection/uhtml)

## Use lit-html

[lit-html](https://github.com/Polymer/lit-html) lets you write HTML templates in JavaScript with template literals.

lit-HTML templates are plain JavaScript. lit-html takes care of rendering templates to DOM, including efficiently updating the DOM with new values.

Below is the Counter code of [using lit-html in the browser](https://apprun-lit-html.glitch.me).

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Hello!</title>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
  </head>
  <body>
  <script type="module">
    import app from 'https://unpkg.com/apprun?module';
    import { render, html } from 'https://unpkg.com/lit-html?module';
    app.render = (e, vdom) => render(vdom, e);
    class Counter extends Component {
      state = 0;
      view = (state) => html`<div>
      <h1>${state}</h1>
        <button @click=${()=>this.run("add", -1)}>-1</button>
        <button @click=${()=>this.run("add", +1)}>+1</button>
      </div>`;
      update =[
        ['add', (state, n) => state + n]
      ]
    }
    new Counter().start(document.body);
  </script>
  </body>
</html>
```

## Use µhtml

[µhtml](https://github.com/WebReflection/uhtml) (micro html) is a ~2.5K lighterhtml subset to build declarative and reactive UI via template literals tags.

```html
<html lang="en">
<head>
  <title>AppRun App</title>
</head>
<body>
  <script type="module">
    import app from 'https://unpkg.com/apprun?module';
    import { render, html } from 'https://unpkg.com/uhtml?module';
    app.render = render;
    class Counter extends Component {
      state = 0;
      view = (state) => html`<div>
      <h1>${state}</h1>
        <button onclick=${() => this.run("add", -1)}>-1</button>
        <button onclick=${() => this.run("add", +1)}>+1</button>
      </div>`;
      update = [
        ['add', (state, n) => state + n]
      ]
    }
    new Counter().start(document.body);
  </script>
</body>
</html>
```

## Child Components

Unlike JSX, you can embed component class into JSX; when using HTML string components, you will need to make a web component/custom element. Then you can embed the components.

```javascript
import app from 'apprun';
import MyComponent from './MyComponent';

app.webComponent('my-component', MyComponent);

const view = state => {
  return `<div>
    <my-component />
  </div>`;
};
app.start('my-app', state, view);
```
