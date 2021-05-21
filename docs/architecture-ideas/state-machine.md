---
title: Use State Machine in AppRun Applications
published: true
description: This post describes creating a state machine in AppRun applications to help event handling using a calculator as an example.
tags: #apprun #javascript #typescript #state_machine
---
## Introduction

The state machine is the tool that developers must have in their toolbox.

> If you are new to the state machine, check out the reference section below.

How can a state machine help?

Typically, when building applications, we follow what's known as the event-driven — where an event happens in the application, we update the application state and render the state to the screen.

Events can happen anytime during user interactions and system interactions, while the application can be in any state. Therefore, before we start to handle the events, we first have to determine the current state and then handle the event accordingly. Sometimes it can be challenging.

The state machine provides a state-event-state mapping. Thus, before we start to handle the events, we know the current state and the future state, so that we only need to focus on the limited state-event scope.

> The specific state machine we are going to use is the [Mealy machine](https://en.wikipedia.org/wiki/Mealy_machine). It has an initial state and then transitions to new states based on events and its current state.

We are going to build a calculator application as an example. You will learn from this post:

* Model a state machine declaratively,
* Make the state machine type-safe
* Add the state machine to the AppRun application

## Model a Calculator

### State and Event

The calculator application looks like this:

```js
const find_transition = (state_machine, state, event) => {
  const current_state = state_machine[state];
  if (!current_state) throw new Error(`No state: ${current_state} found in state machine`);
  const event_tuple = current_state.find(s => s[0] === event);
  return event_tuple ? {
    next_state: event_tuple[1],
    transition: event_tuple[2]
  } : {}
};

const state = {
  _state: 'START',
  display: '0',
  arg1: 0,
  arg2: 0,
  op: '',
  stack: []
};

const view = ({ _state, op, arg1, arg2, display, stack }) => <>
  <style> {`
    .calculator { width: 200px; }
    .buttons {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      grid-gap: 2px;
    }
    button { padding: 10px; width:100%; }
    button:nth-of-type(1) {
      grid-column: span 2;
    }
    button:nth-of-type(16) {
      grid-column: span 2;
    }
  `}
  </style>
  <div class="calculator">
    <h1>{display}</h1>
    <div class="buttons" $onclick={button_click}>
      <button>CE</button>
      <button>+/-</button>
      <button>/</button>
      <button>7</button>
      <button>8</button>
      <button>9</button>
      <button>*</button>
      <button>4</button>
      <button>5</button>
      <button>6</button>
      <button>-</button>
      <button>1</button>
      <button>2</button>
      <button>3</button>
      <button>+</button>
      <button>0</button>
      <button>.</button>
      <button>=</button>
    </div>
    <small>
      {stack.length > 0 && `${stack[0][0]} ${stack[0][1]} `}
      {_state.startsWith("FIRST_") && `${display}`}
      {_state === "OP" && `${arg1} ${op}`}
      {_state.startsWith("SECOND_") && `${arg1} ${op} ${display}`}
      {_state === "EQ" && `${arg1} ${op} ${arg2} = ${display}`}
    </small>
  </div>
</>;

const button_click = (state, e) => {

  const priority = {
    '*': 2,
    '/': 2,
    '+': 1,
    '-': 1
  }

  const getEvent = c => {
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

  let { _state, op, arg1, arg2, display, stack } = state;

  const clear = () => {
    display = '0';
    arg1 = arg2 = 0;
    op = '';
    stack.length = 0;
  }

  const negative = () => {
    display = display.startsWith('-') ? display.substring(1) : '-' + display;
  };

  const calc = () => {
    display = eval(`${arg1}${op}${arg2}`).toString();
  };

  const op1 = () => {
    op = key;
    arg1 = parseFloat(display);
  };

  const op2 = () => {
    if (priority[key] === priority[op]) {
      arg2 = parseFloat(display);
      calc();
      op = key;
      arg1 = parseFloat(display);
    } else if (priority[key] < priority[op]) {
      arg2 = parseFloat(display);
      calc();
      arg1 = parseFloat(display);
      op = key;
      if (stack.length) {
        const f = stack.pop();
        arg1 = eval(`${f[0]}${f[1]}${display}`);
        display = arg1.toString();
      }
    } else {
      stack.push([arg1, op]);
      arg1 = parseFloat(display);
      op = key;
    }

  };

  const eq0 = () => {
    arg1 = parseFloat(display);
    calc();
  };

  const eq2 = () => {
    arg2 = parseFloat(display);
    calc();
    if (stack.length) {
      arg2 = parseFloat(display);
      const f = stack.pop();
      display = eval(`${f[0]}${f[1]}${display}`).toString();
      arg1 = f[0];
      op = f[1];
    }
  };

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

    FIRST_ARG_FLOAT: [
      ['+/-', 'FIRST_ARG_FLOAT', negative],
      ['NUM', 'FIRST_ARG_FLOAT', () => display += key],
      ['OP', 'OP', op1],
      ['CE', 'START', clear]
    ],

    OP: [
      ['NUM', 'SECOND_ARG', () => display = key],
      ['DOT', 'SECOND_ARG', () => display = '0.'],
      ['OP', 'OP', () => op = key],
      ['CE', 'START', clear]
    ],

    SECOND_ARG: [
      ['+/-', 'SECOND_ARG', negative],
      ['NUM', 'SECOND_ARG', () => display += key],
      ['DOT', 'SECOND_ARG_FLOAT', () => display += key],
      ['EQ', 'EQ', eq2],
      ['OP', 'OP', op2],
      ['CE', 'OP', () => display = '0']
    ],

    SECOND_ARG_FLOAT: [
      ['+/-', 'SECOND_ARG_FLOAT', negative],
      ['NUM', 'SECOND_ARG_FLOAT', () => display += key],
      ['EQ', 'EQ', eq2],
      ['OP', 'OP', op2],
      ['CE', 'OP', () => display = '0']
    ],

    EQ: [
      ['+/-', 'FIRST_ARG', negative],
      ['NUM', 'FIRST_ARG', () => display = key],
      ['DOT', 'FIRST_ARG_FLOAT', () => display = '0.'],
      ['EQ', 'EQ', eq0],
      ['OP', 'OP', op1],
      ['CE', 'START', clear]
    ]
  };

  const { next_state, transition } = find_transition(state_machine, _state, event);
  _state = next_state || _state;
  transition && transition();

  return { _state, op, arg1, arg2, display, stack };
}
app.start(document.body, state, view)
```
<apprun-play style="height:350px" no_src="true"></apprun-play>

Click the 'Try the Code' button; you will see the source code.

The calculator has a grid of buttons that users can click at any time. It also displays:

* The numbers that the user typed and the calculation result on top of the grid.
* The calculation formula includes the first argument, the operator, and the second argument, and the calculation result below the gird.

Let's model the initial state of the calculator.

```js
const state = {
  display: '0',
  arg1: 0,
  arg2: 0,
  op: '',
};
```

We handle the buttons' click events in the event handler, _button___click_. Because of the HTML event bubbling, we only need one event handler for all the buttons.

```typescript
const view =
  <div class="buttons" $onclick={button_click}>
  ......
  </div>

const button_click = (state, e) => {
}

app.start(document.body, state, view);
```

That's all we need to do to create an AppRun application, an _initial state_, a _view_, and _event handlers_.

Next, we will add the state machine implementation.

### State Machine

> We follow and extend the calculator state machine from [David's post](https://www.codinglawyer.io/posts/build-javascript-calculator). The post also provides a [diagram](https://www.codinglawyer.io/static/c24440761863d1b3b8bf38e336056726/fa498/sketch-fsm.png) helpful to understand the state machine.

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
You can see the state machine is just a simple data structure.


### Add State-Machine State

We add a new property for tracking the state-machine state, called _*_state*_, into the initial state.

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
const find_transition = <S extends string, E>(
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

Finally, we return a new state from the event handler; AppRun will render the screen accordingly.

Now, we have successfully created the calculator application. You can see the calculator in TypeScript below.

```typescript
export type Transition<T = any> = (state?: T) => void;
export type EventStateTransition<E, S> = [E, S, Transition];
export type StateMachine<S extends string, E> = {
  [key in S]: EventStateTransition<E, S>[];
};
type Events = 'NUM' | 'OP' | 'DOT' | 'CE' | 'EQ' | '+/-';
type States = 'START' | 'FIRST_ARG' | 'FIRST_ARG_FLOAT' | 'OP' | 'SECOND_ARG' | 'SECOND_ARG_FLOAT' | 'EQ';

const find_transition = <S extends string, E>(
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

type Events = 'NUM' | 'OP' | 'DOT' | 'CE' | 'EQ' | '+/-';

type States = 'START' | 'FIRST_ARG' | 'FIRST_ARG_FLOAT' | 'OP' | 'SECOND_ARG' | 'SECOND_ARG_FLOAT' | 'EQ';

const state = {
  _state: 'START' as States,
  display: '0',
  arg1: 0,
  arg2: 0,
  op: '',
  stack: []
};

type State = typeof state;

const view = ({ _state, op, arg1, arg2, display, stack }: State) => <>
  <style> {`
    .calculator { width: 200px; }
    .buttons {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      grid-gap: 2px;
    }
    button { padding: 10px; width:100%; }
    button:nth-of-type(1) {
      grid-column: span 2;
    }
    button:nth-of-type(16) {
      grid-column: span 2;
    }
  `}
  </style>
  <div class="calculator">
    <h1>{display}</h1>
    <div class="buttons" $onclick={button_click}>
      <button>CE</button>
      <button>+/-</button>
      <button>/</button>
      <button>7</button>
      <button>8</button>
      <button>9</button>
      <button>*</button>
      <button>4</button>
      <button>5</button>
      <button>6</button>
      <button>-</button>
      <button>1</button>
      <button>2</button>
      <button>3</button>
      <button>+</button>
      <button>0</button>
      <button>.</button>
      <button>=</button>
    </div>
    <small>
      {stack.length > 0 && `${stack[0][0]} ${stack[0][1]} `}
      {_state.startsWith("FIRST_") && `${display}`}
      {_state === "OP" && `${arg1} ${op}`}
      {_state.startsWith("SECOND_") && `${arg1} ${op} ${display}`}
      {_state === "EQ" && `${arg1} ${op} ${arg2} = ${display}`}
    </small>
  </div>
</>;

const button_click = (state: State, e: any) => {

  const priority = {
    '*': 2,
    '/': 2,
    '+': 1,
    '-': 1
  }

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

  let { _state, op, arg1, arg2, display, stack } = state;

  const clear = () => {
    display = '0';
    arg1 = arg2 = 0;
    op = '';
    stack.length = 0;
  }

  const negative = () => {
    display = display.startsWith('-') ? display.substring(1) : '-' + display;
  };

  const calc = () => {
    display = eval(`${arg1}${op}${arg2}`).toString();
  };

  const op1 = () => {
    op = key;
    arg1 = parseFloat(display);
  };

  const op2 = () => {
    if (priority[key] === priority[op]) {
      arg2 = parseFloat(display);
      calc();
      op = key;
      arg1 = parseFloat(display);
    } else if (priority[key] < priority[op]) {
      arg2 = parseFloat(display);
      calc();
      arg1 = parseFloat(display);
      op = key;
      if (stack.length) {
        const f = stack.pop();
        arg1 = eval(`${f[0]}${f[1]}${display}`);
        display = arg1.toString();
      }
    } else {
      stack.push([arg1, op]);
      arg1 = parseFloat(display);
      op = key;
    }

  };

  const eq0 = () => {
    arg1 = parseFloat(display);
    calc();
  };

  const eq2 = () => {
    arg2 = parseFloat(display);
    calc();
    if (stack.length) {
      arg2 = parseFloat(display);
      const f = stack.pop();
      display = eval(`${f[0]}${f[1]}${display}`).toString();
      arg1 = f[0];
      op = f[1];
    }
  };

  const state_machine: StateMachine<States, Events> = {
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

    FIRST_ARG_FLOAT: [
      ['+/-', 'FIRST_ARG_FLOAT', negative],
      ['NUM', 'FIRST_ARG_FLOAT', () => display += key],
      ['OP', 'OP', op1],
      ['CE', 'START', clear]
    ],

    OP: [
      ['NUM', 'SECOND_ARG', () => display = key],
      ['DOT', 'SECOND_ARG', () => display = '0.'],
      ['OP', 'OP', () => op = key],
      ['CE', 'START', clear]
    ],

    SECOND_ARG: [
      ['+/-', 'SECOND_ARG', negative],
      ['NUM', 'SECOND_ARG', () => display += key],
      ['DOT', 'SECOND_ARG_FLOAT', () => display += key],
      ['EQ', 'EQ', eq2],
      ['OP', 'OP', op2],
      ['CE', 'OP', () => display = '0']
    ],

    SECOND_ARG_FLOAT: [
      ['+/-', 'SECOND_ARG_FLOAT', negative],
      ['NUM', 'SECOND_ARG_FLOAT', () => display += key],
      ['EQ', 'EQ', eq2],
      ['OP', 'OP', op2],
      ['CE', 'OP', () => display = '0']
    ],

    EQ: [
      ['+/-', 'FIRST_ARG', negative],
      ['NUM', 'FIRST_ARG', () => display = key],
      ['DOT', 'FIRST_ARG_FLOAT', () => display = '0.'],
      ['EQ', 'EQ', eq0],
      ['OP', 'OP', op1],
      ['CE', 'START', clear]
    ]
  };

  const { next_state, transition } = find_transition(state_machine, _state, event);
  _state = next_state || _state;
  transition && transition();

  return { _state, op, arg1, arg2, display, stack };
};
app.start(document.body, state, view)
```


## Conclusion

We have created a declarative and type-safe state machine. The state machine data structure is technology agnostic. You can try to use it in React or other frameworks you like. It can naturally fit into AppRun applications.

AppRun is event-driven. Often I feel it is challenging to make events right. Sometimes we define too many events. Sometimes the events come out of order. By using the state machine, I can handle the events within limited state scopes. I have started to think of using more state machines to control the events.

## References

There are many references online about the state machine. I got most of my inspiration from the following posts. I recommend you read the concept explanation of the posts and pay less attention to the implementations because using AppRun; you can do better.

* [1] Krasimir Tsonev explains [Mealy](https://en.wikipedia.org/wiki/Mealy_machine) and [Moore](https://en.wikipedia.org/wiki/Moore_machine) in the post: [The Rise Of The State Machines](https://www.smashingmagazine.com/2018/01/rise-state-machines/)

* [2] Jon Bellah describes the paradigm shift from event-driven to the state machine in this post: [A Complete Introduction to State Machines in JavaScript](https://jonbellah.com/articles/intro-state-machines)

* [3] Erik Mogensen explains state machine and introduced the statechart in this post: [What is a state machine?](https://statecharts.github.io/what-is-a-state-machine.html)

Have fun coding!