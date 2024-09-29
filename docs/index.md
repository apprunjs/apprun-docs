# Welcome to AppRun Docs

Welcome to the AppRun user guide! This guide shows you how to get started creating web applications using [AppRun](https://github.com/yysun/apprun).

## What is AppRun?

AppRun is a lightweight JavaScript library for building applications. It has a unique architecture inspired by the Elm architecture.

Use a _Counter_ as an example.

```js
// define the initial state
const state = 0;

// view is a function to display the state
const view = state => <>
  <h1>{state}</h1>
  <button onclick={()=>app.run('-1')}>-1</button>
  <button onclick={()=>app.run('+1')}>+1</button>
</>;

// update is a collection of event handlers
const update = {
  '+1': state => state + 1,
  '-1': state => state - 1
};

// start the app
app.start(document.body, state, view, update);
```
<apprun-code></apprun-code>

!!! note
    Most of the code snippets in this guide are interactive. Try edit and see the results.

* AppRun is lightweight, only 6KB gzipped, but includes state management, rendering, event handling, and routing.

* With only three functions: `app.start`, `app.run`, and `app.on` in its API makes it easy to learn and use. It can be used directly in the browser or with a compiler/bundler like Webpack or Vite.

* One more thing, you can use AppRun with React to simplify state management and routing of your React applications.


## Quick Start

You can choose one of the following options to get started:

<div class="grid cards" markdown>

-   :material-clock-fast:{ .lg .middle } __Use AppRun in the browser__

    ---

    Reference AppRun from CDN and start building your app in no time

    [:octicons-arrow-right-24: Getting started](tutorial.md)

-   :fontawesome-solid-terminal:{ .lg .middle } __Create an SPA project__

    ---

    Create a new SPA project using the AppRun CLI

    [:octicons-arrow-right-24: Create AppRun App](create-apprun-app.md)

-   :material-react:{ .lg .middle } __Use AppRun with React__

    ---

    Use AppRun with React to simplify state management and routing

    [:octicons-arrow-right-24: React](react.md)

-   :material-scale-balance:{ .lg .middle } __Read More about the architecture__

    ---

    Understand the architecture of AppRun and how it works

    [:octicons-arrow-right-24: Architecture](architecture.md)

</div>



Ready to try it yourself?

Head over to [Getting Started](tutorial.md) or explore the [AppRun Architecture](architecture.md).