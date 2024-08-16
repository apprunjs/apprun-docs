# Use AppRun with React

[React](https://reactjs.org/) is a popular JavaScript library for building user interfaces. Using AppRun and React in conjunction is one of the best ways to build a web app. Here's why you should consider using AppRun with React:

* __Simplified State Management__: React often requires additional libraries like Hooks or Redux for state management. AppRun's state management system is easier use. It can reduce complexity and make your application easier to build and maintain​.

* __Event-Driven Routing__: AppRun's routing system is event-based, treating route changes as events just like any other user interaction. This makes routing in single-page applications (SPA) straightforward and consistent with other app logic. It is a pertect replacement of the React Router or similar tools​​.


* __Seamless JSX Integration__: AppRun supports JSX, the same syntax used by React, which means you can use React as the rendering engine for AppRun components. This allows you to leverage React's powerful rendering capabilities and its vast ecosystem​​.

You can either create an React app and add AppRun to it or create an AppRun app and add React. Here is how you can do it:

## Step 1, Installation


```bash
npm create vite@latest my-app -- --template react
cd my app
npm i apprun
```

or

```bash
npm create apprun-app my-app
cd my-app
npm i react react-dom
```


## Step 2, Setup AppRun

To use React, call AppRun's `app.use_react` function by passing in _React_ and _ReactDOM_.

### For React

```js
import React from 'react';
import ReactDOM from 'react-dom/client'
app.use_react(React, ReactDOM);
```

### For Preact

```js
import React from 'preact/compat';
import ReactDOM from 'preact/compat';
app.use_react(React, ReactDOM);
```

Since Preact is less restrictive than React. You can just use Preact's _render_ function.

```js
import { render } from 'preact'
import app from 'apprun';
app.use_render(render);
```

This way allows us to take advantage of AppRun's unique custom directives ($on, $bind) for handling events and data binding​​.

!!! note
    You need to remember React / Preact uses _onClick_ instead of _onclick_.


## Step 3, Use React JSX and Components

Build your AppRun components using React's JSX in the _view_ function.

```jsx
import { Component } from 'apprun';
import { Button } from 'antd';

export default class ContactComponent extends Component {
  state = 0;

  view = state => (
    <>
      <h1>{state}</h1>
      <Button onClick={() => this.run('add', -1)}>-1</Button>
      <Button onClick={() => this.run('add', +1)}>+1</Button>
      <div><p/>(component)</div>
    </>
  );

  update = {
    add: (state, num) => state + num,
    '#Contact': state => state,
  };
}
```

!!! note
    Since AppRun directives are not supported in React, you need to call _this.run_ to publish events.


## Example

By using React and AppRun together, you can build a powerful web application with AppRun's clean architecture and beautiful React components from [Ant Design](https://ant.design).


Here is an example: https://yysun.github.io/apprun-antd-demo-js

Source: https://github.com/yysun/apprun-antd-demo-js


This project has the [AppRun architecture](architecture.md). As well as the [React components from the Ant Design](https://ant.design/components/overview).


## Conclusion

Whether you add AppRun to your React app or add React to your AppRun app, you can get the best of both worlds.

