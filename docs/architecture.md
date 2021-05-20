# AppRun Architecture

## Architecture Overview

Application logic is broken down into three separated parts in the AppRun architecture.

* State (a.k.a. Model) — the state of your application
* View — a function to display the state
* Update — a collection of event handlers to update the state

Use a _Counter_ as an example.

```js
const state = 0;

const view = state => <div>
  <h1>{state}</h1>
  <button onclick={()=>app.run('-1')}>-1</button>
  <button onclick={()=>app.run('+1')}>+1</button>
</div>;

const update = {
  '+1': state => state + 1,
  '-1': state => state - 1
};

app.start(document.body, state, view, update);
```
<apprun-play></apprun-play>

### State

The _state_ can be any data structure, a number, an array, or an object that reflects the state of the application. In the _ Counter_ example, it is a number.

```js
const state = 0;
```

!!! note
    You define the initial state. AppRun manages the state. Therefore the initial state is an immutable constant.

### View

The _view_ generates Virtual DOM based on the state. Finally, AppRun calculates the differences against the web page element and renders the changes.

```js
const view = state => <div>
  <h1>${state}</h1>
  <button $onclick="-1">-1</button>
  <button $onclick="+1">+1</button>
</div>;
```

!!! note
    AppRun allows you to choose your favorite virtual DOM technology in the _view_ function. The example above uses JSX. You can also use lit-html, uhtml, and etc.

### Update

The _update_ is a collection of named event handlers, or a dictionary of event handlers. Each event handler creates a new state from the current state.
```js
const update = {
  '+1': state => state + 1,
  '-1': state => state - 1
}
```

!!! note
    There are a few other ways to define event handlers.

When the three parts, the _state_, _view_, and _update_ are provided to AppRun to start an application, AppRun registers the event handlers defined in the _update_ and waits for AppRun events.

```js
app.start(document.body, state, view, update);
```

## Architectural Benefits

### Avoid Spaghetti Code

AppRun solves two challenging problems, which I call them:

* Direct State Update
* Rendering Fragments

#### Problem Code

Using a _counter_ example that has two buttons, also, I made the Counter a bitter complicated to show how many times each button clicked.

![Counter](https://dev-to-uploads.s3.amazonaws.com/i/hdjy2w33ipeqdv9wxbea.png)


The code below uses jQuery. jQuery is a library that provides the convenience to access and manipulation the DOM. It does not provide any architectural guidance. jQuery code is similar to the vanilla JavaScript code that can go wild.

```javascript
$(function () {

    // global state
    let count = 0
    let count_plus = 0
    let count_minus = 0

    function plus() {
      // state update
      count ++
      count_plus ++

      // rendering
      $('#total').html(count)
      $('#plus').html(`+ (${count_plus})`)
    }

    function minus() {
      // state update
      count --
      count_minus ++

      // rendering
      $('#total').html(count)
      $('#minus').html(`- (${count_minus})`)
    }

    $('#plus').on('click', plus)
    $('#minus').on('click', minus)

  })
```

You can see from the above code that event handlers _plus_ and _minus_ have the problem patterns. They update the state directly. They also render the DOM in different pieces.

But the real problem is that there isn't a way to break them further. The state has to be shared globally. And the rendering has to be different in each click event.

In much more complicated real applications, the logic could be long and tangled even more.

#### AppRun Code

AppRun is a _state_ management system. It is also an event-driven system that has an event lifecycle. During an AppRun event lifecycle:

* AppRun let you update the _state_ when needed
* AppRun let you create a virtual DOM out of the _state_ when needed
* AppRun renders the virtual DOM when needed.

Following the Hollywood Principle (Don't call us. We call you.) here, we provide code pieces to AppRun and wait for AppRun to call them.

We write functions to update the _state_. First, AppRun gives the _current state_. We create a new _state_ based on the _current state_.

```javascript
const minus = (state) => ({ ...state,
  count: state.count - 1,
  count_minus: state.count_minus + 1
});

const plus = (state) => ({ ...state,
  count: state.count + 1,
  count_plus: state.count_plus + 1
});
```

We will be able to concentrate on the parts that are needed to update. We can spread out the rest of the _state_ using the spread operator. Also, because there is no reference to a shared global object, it is very easy to unit test the _state_ update logic.


We also write a _view_ function that AppRun will call with the _state_ as the input parameter. We usually use JSX in the _view_ function to create a virtual DOM, which is just a data structure. The _view_ function does not render the DOM. AppRun renders the DOM using a diffing algorithm. It only renders the DOM that is needed to change. Therefore, we only need one _view_ function for all events. AppRun takes care of the differential rendering accordingly.

```javascript
const view = ({ count, count_plus, count_minus }) => html`
  <h1>${count}</h1>
  <button onclick="app.run('minus')">- (${count_minus})</button>
  <button onclick="app.run('plus')">+ (${count_plus})</button>`
```

The _view_ function always returns the same result as long as the _state_ is the same. It also does not change the _state_ or anything outside the function, which means it has no side effects. Therefore, we can make the _view_ function a _pure function_. There are many benefits of using _pure function_, including but not limited to unit testing. It makes the UI code easy to unit test.

Using AppRun, we have a _counter_ application made from the _state, _view_, and _update_ as shown below.

```javascript
// initial state object
const state = {
  count: 0,
  count_plus: 0,
  count_minus: 0
}

// one view function to render the state, its' a pure function
const view = ({ count, count_plus, count_minus }) => html`
  <h1>${count}</h1>
  <button onclick="app.run('minus')">- (${count_minus})</button>
  <button onclick="app.run('plus')">+ (${count_plus})</button>
`

// collection of state updates, state is immutable
const plus = (state) => ({ ...state,
  count: state.count - 1,
  count_minus: state.count_minus + 1
});

const minus = (state) => ({ ...state,
  count: state.count + 1,
  count_plus: state.count_plus + 1
});

app.start(document.body, state, view, {plus, minus});
```
<apprun-play></apprun-play>

With the AppRun state management and DOM differential rendering in place, we no longer have the problem mixing **state update** with **DOM rendering**.

#### AppRun Benefits

No matter how complex the application is, we will always have three parts, the _state_, _view_, and _update_. We no longer mix the state update with DOM rendering. Because the three parts are totally decoupled, our codebase is so much easier to understand, test, and maintain.

### Ceremony vs. Essence

There was the 'Ceremony vs. Essence' discussion happened about ten years ago. At that time, Ruby was on the rise. People compared [Ruby with C#](https://davesquared.net/2010/07/essence-and-ceremony-ruby-and-c.html).

> The fundamental idea of the Ceremony vs. Essence idea appears to be that, all other things being equal, programming languages should attempt to allow programmers to clearly express the essence of their programs without being caught up in excessive ceremony provided by the programming language. -- From this [post](http://bryanpendleton.blogspot.com/2010/02/ceremony-vs-essence.html).

Let's take a look at some of today's frontend technologies from the Ceremony vs. Essence point of view. We will use a simple button click counting application as an example.

![Svelte](https://dev-to-uploads.s3.amazonaws.com/i/7mo21xx84dlho7j9tybp.png)

#### The Essence

The essence of the application is to display a button which adds the count by one and shows the count. Also, it will log some messages in the console to mimic effects after the rendering cycle.

The concept is as simple as below.

```javascript
<button onclick="count+1">
  Clicks: {count}
</button>

console.log(count); // upon very click
console.log('mounted!'); // upon mounted
```

We will use the 95-character essence code above with a few frontend frameworks, such as AppRun, Svelte, React Hooks, and the Vue Composition API.

> A framework defines a skeleton where the application defines its features to fill out the skeleton. -- you can find this quote from googling.

When using the frontend frameworks, we need to write some code to plugin the essence code into the frameworks. The code required to plugin into the framework is the ceremony. We don't want them. Less of them is better.

#### The Ceremony

##### AppRun

Application logic is broken down into three separate parts in the AppRun architecture.

* State (a.k.a. Model) — the state of your application
* View — a function to display the state
* Update — a collection of event handlers to update the state

```javascript
import app from 'apprun';

const add = count => count + 1;

const view = count => <button $onclick={add}>
  Clicks: {count}
</button>;

const rendered = count => console.log(count);

app.start(document.body, 0, view, null, { rendered });
console.log('mounted!');
```

In the example above, the application state is a number that has a default value to be 0; the _add_ function is the event handler to update the state. The _view_ function displays the state. The _rendered_ function runs after the DOM is rendered. The _app.start_ function ties them all together to the _document.body_ element.

Now, we identify and cross out the ceremonies.

![AppRun](https://dev-to-uploads.s3.amazonaws.com/i/7eypk2z64h5azepxnlbc.png)

AppRun code ceremony is mainly required by the JavaScript syntax, like the module import and the arrow functions. The only thing needed from AppRun is the _app.start_ function.

Overall, it has 226 characters, which means 58% of code are ceremonies.

##### Svelte

Svelte uses a single file for a component. The file consists of a script section for code and the UI template. It requires a compiler to turn it into the runnable JavaScript code.

```javascript
<script>
  import { onMount } from 'svelte'

  let count = 0;
  const add = () => count + 1;

  $: console.log(count)

  onMount(() => console.log('mounted!')
</script>

<button on:click={add}>
  Clicks: {count}
</button>

```

Behind the scene, the svelte compiler creates the component class boilerplate. Then, the compiler extracts the _script_ block, wires up the [reactivity](https://svelte.dev/tutorial/reactive-assignments) ($:), and adds the rendering template into the boilerplate. The boilerplate does not exist in our codebase. Therefore, the svelte application has very little ceremonies.

![Svelte](https://dev-to-uploads.s3.amazonaws.com/i/o5j864n6r89tvqzqbcev.png)

Svelte code ceremony is also mainly the JavaScript syntax requirements. Only the _script_ tags are required by the Svelte compiler, which is worth of trading with what the compiler saves.

It has 217 characters, which means 56% of code is ceremony.

##### React Hooks

The React code is a slightly modified version from the [React Hooks Docs](https://reactjs.org/docs/hooks-overview.html).

```javascript
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  const add = () => setCount(count + 1)

  useEffect(() => {
    console.log(count);
  });

  return (
    <button onClick={add}>
      Clicks: {count}
    </button>
  );
}
```

The React code has more ceremonies than the AppRun code and Svelte code above. It has 272 characters and a 65% ceremony.

![React](https://dev-to-uploads.s3.amazonaws.com/i/qtmjinxl7zqpoyz0og41.png)

The _setCount, _useState_, and _useEffect_ functions are the code the deal with the React framework itself. They don't help us to express the essence of the application. They are framework ceremonies.

##### Vue Composition API

The Vue code is a slightly modified version of the [Vue Composition API Docs](https://vue-composition-api-rfc.netlify.com/#type-issues-with-class-api).

```javascript
<template>
  <button @click="add">
    Clicks: {{ count }}
  </button>
</template>

import { ref, watchEffect, onMounted } from 'vue'

export default {
  setup() {
    const count = ref(0)
    function add() {
      count.value++
    }

    watchEffect(() => console.log(count.value))

    onMounted(() => console.log('mounted!'))

    return {
      count,
      add
    }
  }
}

```
The Vue code has 355 characters and a 73% ceremony.

![Vue](https://dev-to-uploads.s3.amazonaws.com/i/v44mpss7ysv5wwx4p45j.png)

The _ref_, _watchEffect_, _onMounted_, _setup, _count.value_, and returning an object of _count_, and _add_ are all required by the Vue framework. Sometimes, they may make writing code more difficult.

#### Expression Comparison

We are not stopping at only comparing the character counts. What is more important is how do you express the business logic. How many extra boilerplates are forced to you by the frameworks?

Let's see how do we _increase the counter_ as an example again.

```javascript
// AppRun
const add = counter => counter + 1;

//Svelte
let count = 0;
const add = () => counter + 1;

// React
const [count, setCount] = useState(0);
const add = () => setCount(count + 1);

// Vue
const count = ref(0);
const add = () => count.value++;
```

You can see AppRun uses a _pure function_, which can easily be [strong typed](https://medium.com/@yiyisun/strong-typing-in-apprun-78520be329c1) if you wish. Svelte is also easy to understand. React and Vue are more difficult.

#### Conclusion

Both the AppRun code and the Svelte code express the essence well and have less ceremony. I am biased toward AppRun because I am the author. But, I like the Svelte compiler.

React Hooks and Vue Composition API are cool. However, they both add a lot more ceremonies to our codebase. Remember, the ceremony has no business values but more challenging to understand and, therefore, more costly to maintain.





