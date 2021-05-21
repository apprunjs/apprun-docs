# Directives

AppRun directives are syntax sugars that help simplify the code. They are custom attributes in JSX that have names starting with $. AppRun two out-of-the-box directives: $on and $bind.

## $on...

The $on directive simplifies the code to convert the DOM events to AppRun events.

### Publish Events

```js
class Counter extends Component {
  state = 0;
  view = state => <>
    <h1>{state}</h1>
    <button $onclick='-1'>-1</button>
    <button $onclick='+1'>+1</button>
  </>;
  update = {
    '+1': state => state + 1,
    '-1': state => state - 1
  };
}
new Counter().start(document.body);
```
<apprun-play></apprun-play>


### Invoke Functions

The $on directive can also invoke functions.

```js
class Counter extends Component {
  state = 0;
  view = state => <div>
    <h1>{state}</h1>
    <button $onclick={state=>state-1}>-1</button>
    <button $onclick={state=>state+1}>+1</button>
  </div>;
}
new Counter().start(document.body);
```
<apprun-play></apprun-play>

## $bind

The $bind directive synchronizes the HTML input value to the _state_. See the $bind example below.

```js
const state = '';
const view = state => <div>
  <h1>Hello {state}</h1>
  <input autofocus $bind />
</div>;
app.start(document.body, state, view);
```
<apprun-play></apprun-play>

Or, you can bind to the properties of the _state_.

```js
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

## Custom directive

When AppRun is processing the JSX code, it publishes the $ event when it finds the custom attributes like $X. Thus, you can subscribe to the $ event to provide your directives. E.g., if you create the $animation directive to attach the animation classes from the animation library, [animation.css](https://daneden.github.io/animate.css).

```js
app.on('$', ({key, props}) => {
  if (key === '$animation') {
    const value = props[key];
    if (typeof value === 'string') {
      props.class = \`animated \${value}\`;
    }
  }
});
```
You can see the $animation example below.

```js
// Animation Directive Using animate.css
app.on('$', ({key, props}) => {
  if (key === '$animation') {
    const value = props[key];
    if (typeof value === 'string') {
      props.class = `animated ${value}`;
    }
  }
});

const state = {
  animation: true
}

const start_animation = state => ({ animation: true })
const stop_animation = state => ({ animation: false })

const view = state => <>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.7.0/animate.min.css"></link>
  <img $animation={state.animation && 'bounce infinite'} src='../assets/logo.png' />
  <div $animation='bounceInRight'>
    <button disabled={state.animation} $onclick={start_animation}>start</button>
    <button disabled={!state.animation} $onclick={stop_animation}>stop</button>
  </div>
</>

app.start(document.body, state, view);
```
<apprun-play style="height:230px"></apprun-play>