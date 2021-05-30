# Use AppRun with React

[React](https://reactjs.org/) is a popular JavaScript library for building user interfaces. Using AppRun and React in conjunction is one of the best ways to build a web app.

You can use AppRun with React in one of the two ways.

* Use AppRun components in React apps
* Use React Virtual DOM in AppRun apps

## Use AppRun Components

It only takes one line of code to use AppRun components in React.

Let's use the code from the [React Hooks Doc](https://reactjs.org/docs/hooks-overview.html) as an example.

```js
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

The same app using the AppRun looks like below.

```js
class MyComponent extends Component {
  state = 0;
  view = count => <div>
    <p>You clicked {count} times</p>
    <button onclick={()=>this.run('add')}>
      Click me
    </button>
  </div>;
  update  = {
    add: state => state + 1
  };
}
new MyComponent().start(document.body);

```
<apprun-play></apprun-play>

To use the AppRun component in React is very easy. All you need to do is converting the AppRun component to a React component by calling the _toReact_ function.

```js
import { Component } from 'apprun/esm/component';
import toReact from 'apprun/react';

class MyComponent extends Component {
  state = 0;
  view = count => <div>
    <p>You clicked {count} times</p>
    <button onClick={()=>this.run('add')}>
      Click me
    </button>
  </div>;
  update  = {
    add: state => state + 1
  };
}

const App = toReact(MyComponent);
export default App;
```
!!! note
    The React VDOM uses JSX. AppRun VDOM also uses JSX. They are similar. However, React VDOM does not have directives. So you cannot use the AppRun _$onclick_ directive. Instead, you need to use the React _onClick_ attribute.

Now, with just one line conversion to React component, we successfully used AppRun in React apps. Thus, we have the [elm-inspired AppRun architecture](https://apprun.js.org/docs/architecture/) in the React apps.

You can visit https://github.com/yysun-apprun to see an example React project created by the [Create React App Cli](https://reactjs.org/docs/create-a-new-react-app.html) that uses AppRun components.

## Use React VDOM

On the other hand, since we can use any Virtual DOM (VDOM) technology in AppRun apps, including the one from React, lets' use the React VDOM in AppRun apps.

It also just needs one line of code to replace the _app.render_ function with the _ReactDOM.render_ function.

```js
app.render = (el, vdom) => ReactDOM.render(vdom, el);
```

Below is the AppRun app that uses React VDOM.

```js
import app from 'apprun';
import React from 'react';
import ReactDOM from 'react-dom';

app.render = (el, vdom) => ReactDOM.render(vdom, el);

class MyComponent extends Component {
  state = 0;
  view = count => <div>
    <p>You clicked {count} times</p>
    <button onClick={()=>this.run('add')}>
      Click me
    </button>
  </div>;
  update  = {
    add: state => state + 1
  };
}
new MyComponent().start(document.body);
```

Again, you must remember to use the React _onClick_ attribute instead of the AppRun _$onclick_ directive.

## Conclusion

If you prefer to use React as the main framework, you can use AppRun to get the elm-inspired architecture. If you choose to use AppRun, you can also use the React Virtual DOM for rendering. You get the best of both worlds.

