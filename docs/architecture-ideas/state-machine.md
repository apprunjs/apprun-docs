---
title: Use State Machine in AppRun Applications
published: true
description: This post describes how to create a state machine in AppRun applications to help event handling using a calculator as an example.
tags: #apprun #javascript #typescript #state_machine
---
## Introduction

### State machine

The state machine is the tool that developers must have in their toolbox.

> If you are new to the state machine, check out the reference section below.

How can a state machine help?

Typically, when building applications, we follow what's known as the event-driven — where an event happens in the application, we update the application state and render the state to the screen.

Events can happen anytime during user interactions and system interactions while the application can be in any state. Before we start to handle the events, we first have to determine what is the current state and then handle the event accordingly. Sometimes it can be challenging.

The state machine provides a state-event-state mapping. Before we start to handle the events, we know the current state and the future state, so that we only need to focus on the limited state-event scope.

> The specific state machine we are going to use is the [Mealy machine](https://en.wikipedia.org/wiki/Mealy_machine). It has an initial state and then transitions to new states based on events and its current state.

We are going to build a calculator application as an example. You will learn from this post:

* Model a state machine declaratively,
* Make the state machine type-safe
* Add the state machine to the AppRun application

## Model a Calculator

### State and Event

The calculator application looks like:

![Calculator Application](https://dev-to-uploads.s3.amazonaws.com/i/hfscu8m5zix5dy91b3hw.png)

It has a grid of buttons that users can click at any time. It also displays:

* The numbers that the user types, or the calculation result.
* The calculation formula, which includes the first argument, the operator and the second argument, and the calculation result.

The initial state of the calculator looks like:

```js
const state = {
  display: '0',
  arg1: 0,
  arg2: 0,
  op: '',
};
```

We handle the buttons' click events in the event handler, _button___click_. Because of the HTML event bubbling, we just need one event handler for all buttons

```typescript
const view =
  <div class="buttons" $onclick={button_click}>
  ......
  </div>

const button_click = (state, e) => {
}
```

That's all we need to do to create an AppRun application, an _initial state_, a _view_, and _event handlers_.

Next, we will add a state machine.


### State Machine

We follow and extend the calculator state machine from [David's post](https://www.codinglawyer.io/posts/build-javascript-calculator). The post also provides a [diagram](https://www.codinglawyer.io/static/c24440761863d1b3b8bf38e336056726/fa498/sketch-fsm.png) helpful to understand the state machine.

We first define the _states_ and _events_ of the state machine using TypeScript [Discriminated Unions](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions).

```typescript
type Events = 'NUM' | 'OP' | 'DOT' | 'CE' | 'EQ' | '+/-';

type States =
  'START' |
  'FIRST_ARG' |
  'FIRST_ARG_FLOAT' |
  'OP' |
  'SECOND_ARG' |
  'SECOND_ARG_FLOAT' |
  'EQ';
```

We then define the state machine. It is a collection of all the _states_. Each _state_ has a list of available _events_ and _transitions_ in an array. The _transition_ is the function to update the _state_.

```typescript
const state_machine = {
  START: [
    ['NUM', 'FIRST_ARG', () => display = key],
    ['DOT', 'FIRST_ARG_FLOAT', () => display = '0.']
  ],
  FIRST_ARG: [
    ['+/-', 'FIRST_ARG', negative],
    ['NUM', 'FIRST_ARG', () => display += key],
    ['DOT', 'FIRST_ARG_FLOAT', () => display += key],
    ['OP', 'OP', op1],
    ['CE', 'START', clear]
  ],
   ...
}
```

For example, when the _current state_ is START, and the NUM event comes, the _new state_ should be 'FIRST_ARG (waiting for 1st argument)'. The _display_ property of the _state_ should be the user's input.

Another example, when the _current state_ is FIRST_ARG, and the _+/- event_ comes, the _display_ property should toggle between positive and negative.

So on and so forth. It is straightforward to create the state machine object according to the diagram.

Next, we make the state machine type-safe by adding more types.

```typescript
export type Transition = () => void;
export type EventStateTransition<E, S> = [E, S, Transition];
export type StateMachine<S extends string, E> = {
  [key in S]: EventStateTransition<E, S>[];
};
```

* The _Tansition_ is a function to update the application state.
* The _EventStateTransition_ is a TypeScript [Tuple](https://www.typescriptlang.org/docs/handbook/basic-types.html#tuple). It describes which _event_ leads to which new state.
* The _StateMachine is an object that uses the _States_ as the index key.

Now, the state machine is type-safe. The TypeScript compiler only allows you to use the states and events defined in _States_ and _Events_.

```typescript
const state_machine: StateMachine<States, Events> = {
  START0: [ // Error on START0
    ['NUM0', 'FIRST_ARG', () => {}], // Error on NUM0
    ['DOT', 'FIRST_ARG_FLOAT0', () => {}] // Error on FIRST_ARG_FLOAT0
  ],
}
```

Also, the compiler makes sure all _States_ have their relevant entries in the state machine.

```typescript
const state_machine: StateMachine<States, Events> = {
  START: [],
  FIRST_ARG: [],
  FIRST_ARG_FLOAT: [],
  OP:[], SECOND_ARG:[],
  SECOND_ARG_FLOAT:[],
  //EQ:[] // Error on missing EQ state, if we commented it out
}
```

Compare to many other different ways of implementing the state machine in JavaScript/TypeScript found online, the state machine in this post has the following advantages:

* Declarative - it tells whats, not how;
* Independent - technology stack agnostic;
* KISS - no worry of preconditions, postconditions, etc...

You can see the state machine is just a simple data structure. We can easily add it to the AppRun applications. Explained step by step below.

## Add State Machine to AppRun Application

### Add State Machine State

We add a new property for tracking the state-machine state, called _*_state*_ into the application state.

```js
const state = {
  _state: 'START' as States,
  display: '0',
  arg1: 0,
  arg2: 0,
  op: '',
};
export type State = typeof state;
```

### Convert UI Events

All button clicks use the _button___click_ event handler. We convert UI events into different state-machine events.

```typescript
export const button_click = (state: State, e: any) => {

  const getEvent = (c: string): Events => {
    switch (c) {
      case '+/-':
        return '+/-';
      case 'CE':
        return 'CE';
      case '.':
        return 'DOT';
      case '=':
        return 'EQ';
      default:
        return /\d/.test(c) ? 'NUM' : 'OP';
    }
  };

  const key = e.target?.textContent || e;
  const event = getEvent(key);


}
```

### Use State Machine

Now that we know the current state-machine state from the _*_state*_ property of the application state. We also know which state-machine event we are in. We now can use the _state___machine_ to find the matching _transition_.

Finding _transitions_ from the _state___machine_ is straightforward.

```typescript
export const find_transition = <S extends string, E>(
  state_machine: StateMachine<S, E>,
  state: S,
  event: E
): { next_state?: S, transition?: Transition } => {
  const current_state = state_machine[state];
  if (!current_state) throw new Error(`No state: ${current_state} found in state machine`);
  const event_tuple = current_state.find(s => s[0] === event);
  return event_tuple ? {
    next_state: event_tuple[1],
    transition: event_tuple[2]
  } : {}
};
```

If we found the _transition_, we run the _transition_ function. It updates the destructed application state properties, such as _op_, _arg1_, _arg2_, and _display_ accordingly. We then update the application state to be the _next state_.

```typescript
const button_click = (state, e) => {
  let { _state, op, arg1, arg2, display } = state;
  const event = getEvent(s);
  const state_machine = {
  };

  const { next_state, transition } = find_transition(state_machine, _state, event);
  transition && transition();
  _state = next_state || _state;

  return { _state, op, arg1, arg2, display };
}
```

If no _transition_ found, nothing will happen.

Finally, we return a new state from the event handler, AppRun will render the screen accordingly.

Now, the application is wired up with AppRun architecture. We have successfully created the calculator application.

You can try the live app [here](https://apprun.js.org/#calculator) and find the source code [here](https://github.com/yysun/apprun/blob/master/demo/components/calculator.tsx).

## Conclusion

We have created a declarative and type-safe state machine. The state machine data structure is technology agnostic. You can try to use it in React or other frameworks you like. It can naturally fit into AppRun applications.

AppRun is event-driven. Often I feel it is challenging to make events right. Sometimes we define too many events. Sometimes the events come out of order. By using the state machine, I can handle the events within limited state scopes. I have started to think of using more state machines to control the events.

## References

There are many references online about the state machine. I got most of my inspiration from the following posts. I recommend you read the concept explanation of the posts and pay less attention to the implementations, because using AppRun, you can do better.

* [1] Krasimir Tsonev explains [Mealy](https://en.wikipedia.org/wiki/Mealy_machine) and [Moore](https://en.wikipedia.org/wiki/Moore_machine) in the post: [The Rise Of The State Machines](https://www.smashingmagazine.com/2018/01/rise-state-machines/)

* [2] Jon Bellah describes the paradigm shift from event-driven to the state machine in this post: [A Complete Introduction to State Machines in JavaScript](https://jonbellah.com/articles/intro-state-machines)

* [3] Erik Mogensen explains state machine and introduced the statechart in this post: [What is a state machine?](https://statecharts.github.io/what-is-a-state-machine.html)

Have fun coding!