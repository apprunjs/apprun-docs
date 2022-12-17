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

When AppRun is processing the JSX code, it publishes the $ event when it finds the custom attributes like $X. Thus, you can subscribe to the $ event to provide your directives.

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

### Animation directive

We can create the $animation directive to attach the animation classes from the animation library, [animation.css](https://daneden.github.io/animate.css). See the $animation example below.

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


### Form Validation

HTML5 [Constraint Validation API](https://developer.mozilla.org/en-US/docs/Web/API/Constraint_validation) simplifies client-side validation. We can use basic validation by choosing the type attribute of input elements, such as email and URL. Or we can use the _pattern_ attribute for validation using regular expression. Also, we can use the _required_, _maxlength_, _min_, and _max_ attributes.

We can create the $validation directive to display the validation results by setting and removing the class.

```js
const validate = (e) => {
  const element = e.target;
  element.checkValidity();
  console.log(element.validity.valid);
  if (element.validity.valid) {
    element.classList.remove('is-invalid');
  } else {
    element.classList.add('is-invalid');
  }
}

app.on('$', ({ key, props }) => {
  if (key === '$validate') {
    const event = props[key];
    props['oninput'] = validate;
  }
});


const signIn = (state, e) => {
  e.preventDefault();
  alert('form submitted')
}

const view = ({name}) => <>
  <style>{`.is-invalid { border: 2px solid red; color: red }`}</style>
  <form autocomplete="off" $onsubmit="signIn">
    <p>
    <label for="name" autocomplete="off">Enter an name (letters only): </label>
    <input $validate type="text" name="name" required pattern="[A-Za-z]+"/>
    </p>
    <p>
    <label for="email" autocomplete="off">Enter a email: </label>
    <input $validate type="email" name="email" required />
    </p>
    <p>
    <label for="url" autocomplete="off">Enter an URL: </label>
    <input $validate type="url" name="url" required />
    </p>
    <button type="submit">Submit</button>
  </form>
  <div>{name}</div>
</>;

app.start(document.body, {}, view, {signIn});

```
<apprun-play style="height:230px"></apprun-play>