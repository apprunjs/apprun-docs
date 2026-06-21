# Architecture Practices

AppRun's architecture is the product. Build tools, CSS frameworks, and project scaffolds are replaceable. The part that must stay clear is the runtime contract: state flows into view, events flow into update handlers, and handlers return the next state.

When an AppRun app becomes hard to change, the failure is usually not the bundler. It is one of these:

* state is mutated from several places
* views read hidden external state
* event handlers render DOM fragments directly
* routing is treated as a separate framework instead of an event source
* component lifecycle work has no cleanup

The fix is to restore the AppRun boundaries.

## Keep the Three Parts Separate

AppRun code should make the three responsibilities visible.

```js
const state = {
  count: 0,
  saving: false,
  error: null
};

const view = state => <section>
  {state.error && <p>{state.error}</p>}
  <button disabled={state.saving} $onclick="increment">
    Clicks: {state.count}
  </button>
</section>;

const update = {
  increment: state => ({
    ...state,
    count: state.count + 1
  })
};

app.start(document.body, state, view, update);
```

The view describes the DOM. The update describes the state transition. Neither one should secretly do the other's job.

## State Is a Contract, Not a Dumping Ground

State should describe what the UI can render and what the update handlers need to decide.

Good state is specific:

```js
const state = {
  items: [],
  selectedId: null,
  loadingItems: false,
  saveError: null
};
```

Weak state is vague:

```js
const state = {
  data: {},
  loading: false,
  message: ''
};
```

Use fields such as `loading`, `saving`, `error`, or `successMessage` only when the component actually renders those states. Do not force every component into the same state shape.

## Use Lifecycle Hooks at the Boundary

Use `mounted` when a component embedded in JSX needs to initialize from props, children, or existing state.

```js
class Editor extends Component {
  state = {
    id: null,
    title: ''
  };

  mounted = (props, _children, state) => ({
    ...state,
    id: props.id,
    title: props.title || ''
  });

  view = state => <input $bind="title" />;
}
```

Use `unload` for cleanup. If a component starts a timer, subscription, observer, animation frame, or third-party widget, it owns the cleanup.

```js
class Clock extends Component {
  state = { now: new Date() };

  mounted = () => {
    this.timer = setInterval(() => this.run('tick'), 1000);
  };

  unload = () => {
    clearInterval(this.timer);
  };

  update = {
    tick: state => ({ ...state, now: new Date() })
  };
}
```

Lifecycle hooks are integration points. Keep application rules in update handlers unless the rule only exists because the DOM or host environment exists.

## Use Directives for AppRun Events

AppRun directives keep DOM events connected to AppRun updates.

```js
const update = {
  rename: (state, event) => ({
    ...state,
    name: event.target.value
  }),
  save: state => ({ ...state, saving: true })
};

const view = state => <form>
  <input $bind="name" />
  <textarea $bind="description" />
  <input $oninput="rename" />
  <button $onclick="save">Save</button>
</form>;
```

Use standard DOM events only for DOM-only behavior, such as `stopPropagation`.

```js
const view = state => <div onclick={event => event.stopPropagation()}>
  ...
</div>;
```

Avoid wrapping an AppRun event in an arrow function whose only job is to call `app.run`. That hides the event contract in the view.

```js
// Avoid
<button $onclick={() => app.run('save')}>Save</button>

// Prefer
<button $onclick="save">Save</button>
```

## Model Multi-Step Work Explicitly

Use async generators when an operation has visible intermediate states.

```js
const update = {
  save: async function* (state) {
    if (!state.name.trim()) {
      yield { ...state, error: 'Name is required' };
      return;
    }

    yield { ...state, saving: true, error: null };

    try {
      await api.save(state.name);
      yield { ...state, saving: false };
    } catch (error) {
      yield {
        ...state,
        saving: false,
        error: error.message || 'Save failed'
      };
    }
  }
};
```

This keeps the user-visible lifecycle in one place: validation, loading, success, and failure.

## Treat Routing as Events

Routing should use AppRun's event model before adding another router.

```js
class ProductPage extends Component {
  state = {
    product: null,
    loading: true,
    error: null
  };

  update = {
    '/products': async (state, id) => {
      const product = await api.getProduct(decodeURIComponent(id));
      return { ...state, product, loading: false };
    }
  };
}
```

For links, preserve browser behavior. An AppRun app should not break command-click, control-click, middle-click, external links, downloads, or `target="_blank"`.

```js
const view = product => <a href={`/products/${encodeURIComponent(product.id)}`}>
  Open
</a>;
```

Programmatic navigation is a side effect. Make that obvious by returning nothing.

```js
const update = {
  openProduct: (_state, id) => {
    window.location.href = `/products/${encodeURIComponent(id)}`;
  }
};
```

## Test the Contract

AppRun's architecture is testable because update handlers are plain functions.

```js
test('increment returns the next state', () => {
  expect(update.increment({ count: 1 })).toEqual({ count: 2 });
});
```

For async generators, test the yielded states.

```js
test('save validates before calling the API', async () => {
  const gen = update.save({ name: '', saving: false, error: null });

  const first = await gen.next();
  expect(first.value.error).toBe('Name is required');

  const done = await gen.next();
  expect(done.done).toBe(true);
});
```

For routing, directives, and lifecycle fixes, test the behavior that users feel:

* route events receive the right parameters
* native link behavior is preserved
* `$bind` updates the right state value
* async work cannot overwrite newer state
* `unload` releases resources

The goal is not coverage theater. The goal is to protect the architectural boundary that made the code simple in the first place.
