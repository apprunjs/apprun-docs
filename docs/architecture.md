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

The _update_ is a collection of named event handlers or a dictionary of event handlers. Each event handler creates a new state from the current state.
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

Next, let's review some of the benefits of AppRun Architecture.


## Avoid Spaghetti Code

AppRun solves two challenging problems commonly found in application development:

Let's make the _Counter_ a bitter complicated to show how many times each button clicked.

```js
--8<-- "counter.js"
```
<apprun-play hide_src="true" hide_button="true"></apprun-play>

The code below uses jQuery. jQuery is a library that provides the convenience to access and manipulation the DOM. It does not give any architectural guidance. jQuery code is similar to the vanilla JavaScript code that can go wild.

### An jQuery Example

```js
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

You can see from the above code that

* The state is shared globally. The two event handlers _plus_ and _minus_ both update the state directly.
* The two event handlers also both render the DOM in different pieces.

Therefore, the jQuery code has two problems:

* Direct State Update
* Rendering Fragments

In much more complicated real applications, the logic could be long and tangled even more.

How can we solve the problems using AppRun?

### AppRun Code

AppRun includes the _state_ management, event system, and Virtual-DOM rendering. Following the Hollywood Principle (Don't call us. We call you), we provide code pieces to AppRun and wait for AppRun to call them.

During an AppRun event lifecycle:

* AppRun let you update the _state_ when needed
* AppRun let you create a virtual DOM out of the _state_ when needed
* AppRun renders the virtual DOM when needed.

When using AppRun to update _state_, AppRun gives the _current state_. Then, we create a new _state_ based on the _current state_.

```js
const minus = (state) => ({ ...state,
  count: state.count - 1,
  count_minus: state.count_minus + 1
});

const plus = (state) => ({ ...state,
  count: state.count + 1,
  count_plus: state.count_plus + 1
});
```

Because there is no reference to a shared global object, it is very easy to unit test the logic. Also, we can focus on the parts of _state_ that are needed to update and ignore the rest of the _state_ using the spread operator.

We only write a _view_ function that creates a virtual DOM. AppRun renders the DOM using the diffing algorithm. It only updates the DOM that is needed to change. Therefore, although we have only one _view_ function for all events, AppRun takes care of the differential rendering accordingly.

```js
const view = ({ count, count_plus, count_minus }) => html`
  <h1>${count}</h1>
  <button onclick="app.run('minus')">- (${count_minus})</button>
  <button onclick="app.run('plus')">+ (${count_plus})</button>`
```

The _view_ function always returns the same result as long as the _state_ is the same. It also does not change the _state_ or anything outside the function, which means it has no side effects. Therefore, the _view_ function is a _pure function_. There are many benefits of using _pure function_, including but not limited to unit testing.

Finally, We have a _ counter _ application shown below by putting the _state, _view_, and _update_ together.

```js
--8<-- "counter.js"
```
<apprun-play></apprun-play>

You can see that with the help of AppRun state management and DOM differential rendering, we no longer have the **Direct State Update** with **Rendering Fragments** problems.

### AppRun Benefits

No matter how complex the application is, we will always have three parts, the _state_, _view_, and _update_. We don't mix the state update with DOM rendering. Because the three parts are decoupled, our codebase is so much easier to understand, test, and maintain.

## Ceremony vs. Essence

There was the 'Ceremony vs. Essence' discussion happened about ten years ago. At that time, Ruby was on the rise. So people compared [Ruby with C#](https://davesquared.net/2010/07/essence-and-ceremony-ruby-and-c.html).

> The fundamental idea of the Ceremony vs. Essence idea appears to be that, all other things being equal, programming languages should attempt to allow programmers to clearly express the essence of their programs without being caught up in excessive ceremony provided by the programming language. -- From this [post](http://bryanpendleton.blogspot.com/2010/02/ceremony-vs-essence.html).

Let's take a look at some of today's frontend technologies from the Ceremony vs. Essence point of view. We will use a simple button click counting application as an example.

```js
const add = count => count + 1;

const view = count => <button $onclick={add}>
  Clicks: {count}
</button>;

const rendered = count => console.log(count);

app.start(document.body, 0, view, null, { rendered });
console.log('mounted!');
```
<apprun-play hide_src="true" hide_button="true"></apprun-play>

### The Essence

The essence of the application is to display a button that adds the count by one and shows the count. Also, it will log some messages in the console to mimic effects after the rendering cycle.

The concept is as simple as below.

```js
<button onclick="count+1">
  Clicks: {count}
</button>

console.log(count); // upon very click
console.log('mounted!'); // upon mounted
```

We will compare the 95-character essence code above with a few frontend frameworks, such as AppRun, Svelte, React Hooks, and the Vue Composition API.

> A framework defines a skeleton where the application defines its features to fill out the skeleton. -- you can find this quote from googling.

We need to write code to plugin the essence code into the frontend frameworks, which is the ceremony. We don't want them. Less of them is better.

### The Ceremony

#### AppRun

Application logic is broken down into three separate parts in the AppRun architecture.

```js
const add = count => count + 1;

const view = count => <button $onclick={add}>
  Clicks: {count}
</button>;

const rendered = count => console.log(count);

app.start(document.body, 0, view, null, { rendered });
console.log('mounted!');
```
<apprun-play></apprun-play>

In the example above,

1. The application's state is a number that has a default value of 0.
2. The _add_ function is the event handler to update the state.
3. The _view_ function displays the state.
4. The _rendered_ function runs after the DOM is rendered.
5. The _app.start_ function ties them all together to the _document.body_ element.

Now, we identify and cross out the ceremonies.

![AppRun](https://dev-to-uploads.s3.amazonaws.com/i/7eypk2z64h5azepxnlbc.png)

With AppRun, the ceremony is mainly required by the JavaScript syntax, like the module import and the arrow functions. Overall, it has 226 characters, which means 58% of the code are ceremonies.

#### Svelte

Svelte uses a single file for a component. The file consists of a script section for code and the UI template. It requires a compiler to turn it into the runnable JavaScript code.

```js
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

Behind the scene, the svelte compiler creates the component class boilerplate. Then, the compiler extracts the _script_ block, wires up the [reactivity](https://svelte.dev/tutorial/reactive-assignments) ($:), and adds the rendering template into the boilerplate. The boilerplate does not exist in our codebase. Therefore, the svelte application has very few ceremonies.

![Svelte](https://dev-to-uploads.s3.amazonaws.com/i/o5j864n6r89tvqzqbcev.png)

Svelte code ceremony is also mainly the JavaScript syntax requirements. Only the _script_ tags are required by the Svelte compiler, which is worth trading with what the compiler saves.

It has 217 characters, which means 56% of the code is ceremony.

#### React Hooks

The React code is a slightly modified version from the [React Hooks Docs](https://reactjs.org/docs/hooks-overview.html).

```js
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

The _setCount, _useState_, and _useEffect_ functions are the code that deals with the React framework itself. They don't help us to express the essence of the application. They are framework ceremonies.

#### Vue Composition API

The Vue code is a slightly modified version of the [Vue Composition API Docs](https://vue-composition-api-rfc.netlify.com/#type-issues-with-class-api).

```js
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

### Expression Comparison

We are not stopping at only comparing the character counts or how many extra boilerplates are forced on you by the frameworks. We also compare how do you express the business logic. For example, let's see how we express _**Increase the Counter**_ as an example again.

```js
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

Both the AppRun code and the Svelte code express the essence well and have less ceremony. So AppRun and Svelte are easy to understand. React Hooks and Vue Composition API are cool. However, they both add a lot more ceremonies to our codebase.

Remember, the ceremony has no business values but just challenges to understand and maintain.

In addition, AppRun has a few other benefits

* AppRun is lightweight that can run in browsers directly without a compiler.
* AppRun uses _pure functions_ when it is possible.
* AppRun app codebase can easily be [strong typed](strong-typing.md) if you wish.

I hope you enjoy it. If you haven't clicked the 'Try the Code' buttons to run the AppRun code above, please give it a try.

