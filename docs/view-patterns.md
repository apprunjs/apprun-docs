# JSX

AppRun uses virtual DOM technology (VDOM). The VDOM is the data representing a DOM structure. AppRun compares the VDOM with the real DOM and updates only the changed elements and element properties. It provides high performance.

AppRun allows you to choose your favorite virtual DOM technology to create user interfaces in the _view_ function.

We recommend using JSX. Some advanced features only apply to JSX.

## JSX

JSX is a syntax sugar of function calls. You can compose the functions and apply dynamic and conditional rendering without the run-time cost of parsing the HTML string.


You can use the JSX features described below.

### JSX fragments

JSX Fragments let you group a list of children without adding extra root node. E.g., you can use <></> for declaring fragments. E.g.,

```javascript
const view = <>
  <h1>title 1</h1>
  <h2>title 2</h2>
</>
```

### Function Calls

We can also use the capitalized JSX tag to call JavaScript functions with capitalized function names. The functions are also known as the Pure Function Component.

E.g., To render the todo item list, You can call the Todo function in an array.map function.

```javaScript
const Todo = ({ todo, idx }) => <li>{todo.title}</li>;
const view = state => <ul class="todo-list"> {
  state.list.map((todo, idx) => <Todo todo={todo} idx={idx} />)
}</ul>
```

### De-structuring Properties
The call to the Todo function passes two properties todo and idx. In the Todo function, you can retrieve the two properties by de-structuring the parameters.

```javascript
const Todo = ({ todo, idx }) => <li>{todo.title}</li>;
```

### Set Class

Each todo item should have class “view” represents that active or complete for a complete status of the todo item. You can use the ternary operator to toggle between two classes.

```javascript
const Todo = ({ todo, idx }) => <li class={todo.done ? "completed" : "view"}>
```

Please note that AppRun supports using the keyword _**class**_ in JSX.

### Toggle Class

Sometimes, you need to toggle classes based on the state. You can also use the ternary operator to toggle the class. E.g., toggle the _selected_ class to a menu item.

```javascript
<li><a class={state.filter === 'All' ? 'selected' : ''} >All</a></li>
```

### Show and Hide Element

To show or hide an element dynamically, you can use the _&&_ operator.

```javascript
const countComplete = state.list.filter(todo => todo.done).length || 0;
{ countComplete && <button>Clear completed</button>}
```

### _ref_

[_ref_](https://apprun.js.org/#play/12) is a special JSX property, which is a callback function that is called after the _view_ function is executed.

```javascript
const view = <div ref={el=>{...}}></div>
```

We can use _ref_ function to update the HTML element, e.g., [set focus to an input box](https://apprun.js.org/#play/12).

_ref_ is a better method to update the element than using the _rendered_ lifecycle function.

>Please think of using the _ref_ function before you use the _rendered_ function.


### Element embedding

Furthermore, AppRun allows [embedding elements directly into JSX](https://apprun.js.org/#play/14).

```javascript
view = state => {
  const canvas = document.createElement('canvas');
  return <div>{canvas}</div>
};
```

A few use cases of the _Element embedding_ are:

* Create special element, e.g. [element has shadow root](https://apprun.js.org/#play/16)
* Create elements using 3rd libraries.
* Create and cache the element to avoid recreation in every event lifecycle

Just create the HTML element and add it to the AppRun _view_.

>Please think of embedding the element before you use the _ref_ function.

### Directive

The directive is the special property that looks like $xxx. When AppRun is processing the JSX code and finds the properties of $xxx, it publishes the $ event. The event parameters contain the directive key, properties, and tag Name of the HTML element and component instance.

```javascript
const view = <div $myDirective></div>;
app.on('$', ({key, props, tag, component}) => {
  if (key === '$myDirective') {
  }
}
```

We can subscribe to the $ event and create _custom directives_ to modify the properties of the HTML element, e.g., [adding or removing classes](https://apprun.js.org/#play/11).



Next, you will need to learn how to handle users' navigation and activate the components, which is also known as [routing](07-routing).

