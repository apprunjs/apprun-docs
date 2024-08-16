# Quick Start Tutorial

## Create Your First App

AppRun Application logic is broken down into three separate parts in the AppRun architecture.

* State (a.k.a. Model) — the state of your application
* View — a function to display the state
* Update — a collection of event handlers to update the state

Let's use the _Counter_ app as an example and code it directly in the HTML file.

```html
<html>
<body>
  <script src="https://unpkg.com/apprun/dist/apprun-html.js">
  </script>
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
<apprun-code></apprun-code>

!!! note
    Browsers dont support JSX, so we use the _html_ function (Template literals) in the _view_ function to create HTML elements.


## Create Your First Component

AppRun components are mini-applications with the _state_, _view_, and _update_ architecture.

Let's re-create the _Counter_ app as a component.

```html
<html>
<body>
  <script src="https://unpkg.com/apprun/dist/apprun-html.js"></script>
  <script>
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
  </script>
</body>
</html>
```
<apprun-code></apprun-code>

!!! note
    Components have local event events. We use _this.run_ instead of _app.run_ to publish local events.


## Use Components for SPA

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
[About, Contact, Home]
  .map(C => new C().start('pages'));
```
<apprun-code code-width="50%"></apprun-code>

!!! note
    We have just created a simple SPA using components. In a real-world scenario, usually, create pages as modules and bundle them together or load them dynamically. See the [Single Page App](spa.md) for more details.


## Create a Web Component

AppRun components can be defined as web components/custom elements and used in HTML. All we need to do is to give the AppRun component a custom-element name using the _app.webComponent_ function.

```js
<html>
<head>
  <meta charset="utf-8">
  <title>Counter</title>
</head>
<body>
  <my-counter></my-counter>
  <my-counter></my-counter>
  <my-counter></my-counter>
  <script src="https://unpkg.com/apprun/dist/apprun-html.js">
  </script>
  <script>
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
  </script>
</body>
</html>

```
<apprun-code></apprun-code>

!!! note
    We have create a web component / custom element and used it three times.

<br />

Now, you are ready to move forward to more advanced topics:



<div class="grid cards" markdown>

-   :material-clock-fast:{ .lg .middle } __Use AppRun as ESM in browser__

    ---

    Reference AppRun from CDN as ESM to build your apps

    [Use AppRun Module in the browser](esm.md)

-   :fontawesome-solid-terminal:{ .lg .middle } __Create an SPA project__

    ---

    Create a new SPA project using the AppRun CLI

    [:octicons-arrow-right-24: Create AppRun App](create-apprun-app.md)

</div>



