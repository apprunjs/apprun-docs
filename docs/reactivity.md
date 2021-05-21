---
title: Reactivity in AppRun
published: true
description: This post describes the reactive implementation in AppRun
tags: #apprun #vue #svelte #angular
---

## Introduction

Reactivity has been a hot buzzword for modern JavaScript UI frameworks in the past few years. Angular, Vue, and Svelte all have reactivity built-in. They are famous and popular because of their reactivity features.

Reactivity means that the changed application state will automatically reflect in the DOM.

> Don't be confuse the _reactivity_ with another buzz word, _reactive programming_. Reactive programming is programming with asynchronous data streams. I will have another post to explain _reactive programming_.

Reactivity is related to the _data binding_ concept. _Data binding_ is the process that establishes a connection between the application state and the application UI. There are two major types of _data binding_: _one-way bing_ and _two-binding_.

* _One-way binding_ means that changes in the application state cause changes to the application UI.

* _Two-way binding_ means that either application state or application UI changes (for example, with input elements) automatically update the other.

The reactivity also applies to the state object properties. E.g., if there is a person object with the properties of first-name, last-name, and full-name, we want the full-name property to be reactive to the other two name properties.

With the _reactivity_ concept clarified, let's see how we can have _reactivity_ in AppRun.

## One-Way

Many frameworks use the concept of "variable assignments trigger UI updates." E.g., [Vue](https://v1.vuejs.org/guide/reactivity.html) wires up the application _state_ objects with a change detection mechanism to become a view model or Proxy. Then you can modify the view model to trigger the UI update. [Svelte](https://github.com/sveltejs/rfcs/blob/master/text/0001-reactive-assignments.md) has a compiler to inject change detection around your application state object. You can also modify the state to trigger the UI update.

Unlike other frameworks, AppRun uses the [events](https://observablehq.com/@yysun/apprun-events-evolved) to trigger UI updates following the event-driven web programming model naturally. During an AppRun _event lifecycle_:

* AppRun gives you the current _state_ for you to create a new _state_
* AppRun calls your _view_ function to create a virtual
* AppRun renders the virtual DOM if it is not null.

You can feel the [Hollywood Principle](https://wiki.c2.com/?HollywoodPrinciple) (Don't call us. We call you.) here, which usually means things are loosely coupled. We provide code pieces. The framework calls them when needed.

In the example below, the AppRun $onclick directive calls the event handler, then calls the view function, and then renders the virtual DOM.

```js
const view = state => <div>
  <h1>{state}</h1>
  <button $onclick={state => state - 1}>+1</button>
  <button $onclick={state => state + 1}>+1</button>
</div>;

app.start(document.body, 0, view)
```
<apprun-play></apprun-play>

## Two-Way Binding

AppRun $bind directive can update the _state_ properties automatically when used with the _input_ elements and the _textarea_ element. It looks similar to Angular's *ngModel*, Vue' *v-model*, and Svelte's *bind:value* syntax. However, Angular, Vue, and Svelte have invented their own proprietary template language/syntax that you need to learn. AppRun uses the JSX that React also uses.

```js
const view = state => <>
  <div>{state.text}</div>
  <input $bind="text" placeholder="type something here ..."/>
</>
app.start(document.body, {}, view)
```
<apprun-play></apprun-play>

## Reactive State

The state properties' reactivity is not a problem that the UI frameworks are to solve. But if the UI frameworks wrap or change the original _state_ objects, they have to solve the reactivity problems. E.g., Vue uses the [_computed object_](https://vuejs.org/v2/guide/computed.html). Svelte uses the [reactive-declarations](https://svelte.dev/examples#reactive-declarations), the famous *$:* sign.


### Property Getter

Like in languages like Java and C#, JavaScript has [object property getter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get), which we can use to compute the property values dynamically.

```js
const state = ({
  a: 1,
  b: 2,
  get c() {
    return this.a + this.b;
  }
})
```

Binding to the _state_ object properties is straightforward.

```js
// Reactivity - getter
const state = {
  a: 1,
  b: 2,
  get c() {
    return this.a + this.b;
  }
};
const view = ({a, b, c}) => <>
  <input type="number" $bind="a" />
  <input type="number" $bind="b" />
  <p>{a} + {b} = { c }</p>
</>;
app.start(document.body, state, view);
```
<apprun-play></apprun-play>

### ES2015 Proxy

The [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) is used to define custom behavior for fundamental operations (e.g., property lookup, assignment, enumeration, function invocation, etc.).

> Proxies enable you to intercept and customize operations performed on objects (such as getting properties). They are a _metaprogramming_ feature. - from [Metaprogramming with proxies](https://exploringjs.com/es6/ch_proxies.html#sec_programming-vs-metaprogramming)

To create a Proxy, we create a handler first. Then, we combine the object proxied with the handler.

```js
const handler = ({
  get: (target, name) => {
    const text = target.text || '';
    switch (name) {
      case 'text': return target.text;
      case 'characters': return text.replace(/\s/g, '').length;
      case 'words': return !text ? 0 : text.split(/\s/).length;
      case 'lines': return text.split('\n').length;
      default: return null
    }
  }
})

const state = new Proxy(
  { text: "let's count" },
  handler
)
```

Proxy has almost no barrier to use. Anywhere accepts objects can use Proxy. For example, AppRun can take a _state_ with Proxy.

```js
// Reactivity - Proxy
const handler = {
  get: (target, name) => {
    const text = target.text || '';
    switch (name) {
      case 'text': return target.text;
      case 'characters': return text.replace(/\s/g, '').length;
      case 'words': return !text ? 0 : text.split(/\s/).length;
      case 'lines': return text.split('\n').length;
      default: return null
    }
  }
};
const state = new Proxy(
  { text: "let's count" },
  handler
);
const view = state => <div>
  <textarea rows="10" cols="50" $bind="text"></textarea>
  <div>chars: {state.characters} words: {state.words} lines: {state.lines}</div>
  <pre>{state.text}</pre>
</div>;
app.start(document.body, state, view);
```
<apprun-play style="height:300px"></apprun-play>

I like Proxy because it takes the property value calculation logic out of the _state_ objects. As a result, the _proxy handler_ is much easier to test and maintain. On the other hand, the _state_ objects stay lean. I want the [_state_ to act like the _data transfer object_ (DTO)](https://apprun.js.org/docs/#/04-architecture) in traditional multi-layered application architecture, where the DTO is an object that carries data between logical and physical layers.


## Conclusion

AppRun has full reactivity support that provides us the one-way and two-way data binding and the reactive _state_. Thus, we only need to use the native JavaScript/TypeScript features. Furthermore, AppRun does not require you to learn a new language or a new templating syntax.






