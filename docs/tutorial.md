# Quick Start Tutorial

## Create Your First App

AppRun Application logic is broken down into three separated parts in the AppRun architecture.

* State (a.k.a. Model) — the state of your application
* View — a function to display the state
* Update — a collection of event handlers to update the state

Let's use the _Counter_ app as an example and code it directly in the HTML file.

```html
<html>
<head>
  <meta charset="utf-8">
  <title>Counter</title>
</head>
<body>
  <script src="https://unpkg.com/apprun/dist/apprun-html.js"></script>
  <script>
    const state = 0;
    const view = state => {
      return html`<div>
        <h1>${state}</h1>
        <button onclick='app.run("-1")'>-1</button>
        <button onclick='app.run("+1")'>+1</button>
      </div>`;
    };
    const update = {
      '+1': state => state + 1,
      '-1': state => state - 1
    };
    app.start(document.body, state, view, update);
  </script>
</body>
</html>
```
<apprun-play></apprun-play>

It is easy to have simple code in the HTML file. However, most of the time, we use external script files for complex app logic.

```html
<html>
<head>
  <meta charset="utf-8">
  <title>Counter</title>
</head>
<body>
  <script src="https://unpkg.com/apprun/dist/apprun-html.js"></script>
  <script src="app.js"></script>
</body>
</html>
```

!!! note
    We will use the script file from now on in this tutorial.


## Create Your First Component

AppRun components are mini-applications with the  _state_, _view_, and _update_ architecture.

To create a component, the component class extends the AppRun Component class and defines the  _state_, _view_, and _update_ properties/fields.

```js
class Counter extends Component {
  state = '';
  view = state => <div/>;
  update = {};
}
```

It is straightforward to re-create the _Counter_ app as a component.


```js
//app.js
class Counter extends Component {
  state = 0;
  view = state => {
    return html`<div>
      <h1>${state}</h1>
      <button @click=${()=>this.run("-1")}>-1</button>
      <button @click=${()=>this.run("+1")}>+1</button>
    </div>`;
  };
  update = {
    '+1': state => state + 1,
    '-1': state => state - 1
  };
}
new Counter().start(document.body);

```
<apprun-play></apprun-play>

!!! note
    Components have local event events. We use _this.run_ instead of _app.run_ to publish local events.


## Create a Web Component

AppRun components can be defined as web components/custom elements and used in the HTML.

```html
<html>
<head>
  <meta charset="utf-8">
  <title>Counter</title>
</head>
<body>
  <my-counter></my-counter>
  <script src="https://unpkg.com/apprun/dist/apprun-html.js"></script>
  <script src="app.js"></script>
</body>
</html>
```

All we need to do is to give AppRun component a custom-element name.

```js
//app.js
class Counter extends Component {
  state = 0;
  view = state => {
    return html`<div>
      <h1>${state}</h1>
      <button @click=${()=>this.run("-1")}>-1</button>
      <button @click=${()=>this.run("+1")}>+1</button>
    </div>`;
  };
  update = {
    '+1': state => state + 1,
    '-1': state => state - 1
  };
}
app.webComponent('my-counter', Counter);
document.body.insertAdjacentHTML('beforeend',
`<my-counter></my-counter>
 <my-counter></my-counter>
 <my-counter></my-counter>
`);

```
<apprun-play style="height:350px"></apprun-play>



## Create a Single-Page App

We can easily make a single-page page (SPA) using AppRun components. Each page is a component that can be activated by anchor links like #Home, #contact, and #about.

```js
class Home extends Component {
  view = () => <div>Home</div>;
  update = {'#, #home': state => state };
}

class Contact extends Component {
  view = () => <div>Contact</div>;
  update = {'#contact': state => state };
}

class About extends Component {
  view = () => <div>About</div>;
  update = {'#about': state => state };
}

const App = () => <>
  <div id="menus">
    <a href="#home">Home</a>{' | '}
    <a href="#contact">Contact</a>{' | '}
    <a href="#about">About</a></div>
  <div id="pages"></div>
</>

app.render(document.body, <App />);
[About, Contact, Home].map(C => new C().start('pages'));
```
<apprun-play></apprun-play>

<br />

!!! note
    We have just created a simple SPA using components. In a real-world scenario, usually, we create pages as modules and bundle them together or load them dynamically.

Please continue to read the next section of this guide to learn more about [SPA, SSR, and static site](apprun-site.md).
