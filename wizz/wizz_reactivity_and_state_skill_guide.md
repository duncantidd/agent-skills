# Wizz Native — Reactivity, State & Lifecycle

## Reactivity & State Decision Matrix

| Goal | Tool / Construct | Why |
| :--- | :--- | :--- |
| Local reactive state | Top-level `let count = 0;` | Mutating `let` triggers batched DOM reconciliation scheduled on the next microtask. |
| Accept parent data | `export let propName;` | Declares reactive, read-only component prop passed as attribute by parent. |
| Cross-tab persistent state | `let theme = persist("theme_key", "dark");` | Top-level reactive state backed by `localStorage`; synchronizes across browser tabs automatically. |
| Run setup after DOM attach | `onMount(() => { ... })` | Built-in lifecycle hook; executes after component is attached and children are mounted. |
| Cleanup timers/listeners | `onDestroy(() => { ... })` | Built-in lifecycle hook; executes immediately before component teardown. |
| Handle native browser events | `on:<event>={handler}` | Native DOM event binding without synthetic event wrapper overhead. |

---

## Local Reactive State & Microtask Batching

Any top-level variable declared with `let` inside the `<script>` block is treated as reactive state:

- Reassignments (`count = count + 1`, `items = [...items, newItem]`) schedule a DOM update.
- Multiple state mutations within the same synchronous tick or event handler are batched and applied in a single DOM update on the next microtask.

```html
<!-- src/components/Counter.wizz -->
<div class="counter-box">
  <p class="display">Value: <strong>{count}</strong></p>
  <button type="button" on:click={increment}>Increment</button>
  <button type="button" on:click={reset}>Reset</button>
</div>

<script>
  let count = 0;

  function increment() {
    count = count + 1;
  }

  function reset() {
    count = 0;
  }
</script>
```

---

## Component Props (`export let`)

Props allow parent components to pass data into child components:

- **Declaration**: Use `export let propName;` or `export let propName = defaultValue;`.
- **Read-Only**: Props are strictly read-only within the child component. Do not reassign prop variables inside the child.
- **Dynamic Updates**: If the parent component's state changes, updated prop values propagate reactively into the child.

```html
<!-- Child component: src/components/MetricDisplay.wizz -->
<div class="metric">
  <span class="label">{label}</span>
  <span class="value">{value}</span>
</div>

<script>
  export let label = "Metric";
  export let value = 0;
  // DO NOT: value = 10; (Props are read-only)
</script>
```

---

## Persistent Reactive State (`persist`)

Wizz provides a native persistence mechanism that avoids manual `localStorage.getItem` / `setItem` boilerplate:

- **Declaration**: Must be a **top-level** `let` declaration initialized with `persist(storageKey, defaultValue)`.
- **Automatic Storage**: Reassigning the variable updates both the DOM and `localStorage`.
- **Cross-Tab Synchronization**: Automatically listens for `storage` events; changes in one tab reactively update all open tabs sharing the same key.

```html
<!-- src/components/ThemeToggle.wizz -->
<div class="toggle-container">
  <button type="button" on:click={toggleTheme}>
    Mode: {currentTheme}
  </button>
</div>

<script>
  // Top-level declaration persists across page reloads and browser tabs
  let currentTheme = persist("wizz_app_theme", "light");

  function toggleTheme() {
    currentTheme = currentTheme === "light" ? "dark" : "light";
  }
</script>
```

---

## Lifecycle Management: `onMount` & `onDestroy`

Wizz provides global lifecycle hooks available directly within `<script>` without requiring imports:

- **`onMount(callback)`**:
  - Runs on the client immediately after the component is mounted to the live DOM and its children are initialized.
  - Safe place to initialize `AbortController`, start WebSockets, or measure DOM nodes.
  - State assignments inside statement-level hook callbacks trigger reactive updates.
- **`onDestroy(callback)`**:
  - Runs immediately prior to DOM detachment.
  - Essential for clearing `setInterval`, tearing down external listeners, and aborting active fetch controllers.

```html
<!-- src/components/LiveClock.wizz -->
<div class="clock-display">
  <time>{currentTime}</time>
</div>

<script>
  let currentTime = new Date().toLocaleTimeString();
  let timerId = null;

  onMount(() => {
    timerId = setInterval(() => {
      currentTime = new Date().toLocaleTimeString();
    }, 1000);
  });

  onDestroy(() => {
    if (timerId !== null) {
      clearInterval(timerId);
    }
  });
</script>
```

---

## Event Handling Patterns

Events are bound to native HTML elements using the `on:<event>={handler}` directive:

- Event names correspond directly to native browser events (`click`, `input`, `submit`, `keydown`).
- Handlers must be named functions defined in `<script>`.
- Inline arrow functions (e.g. `on:click={() => count++}`) are not supported by the template syntax.

```html
<!-- src/components/SearchBox.wizz -->
<div class="search-wrap">
  <input
    type="search"
    placeholder="Filter items..."
    value="{query}"
    on:input={handleInput}
  />
  <button type="button" on:click={clearSearch}>Clear</button>
</div>

<script>
  let query = "";

  function handleInput(event) {
    query = event.target.value;
  }

  function clearSearch() {
    query = "";
  }
</script>
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| Mutating `export let` props inside child (`export let count; count = 5;`) | Notify parent or manage local copy (`let localCount = count;`) | Props are strictly read-only in child components; mutation causes compiler error or desynchronization. |
| Importing lifecycle hooks (`import { onMount } from 'wizz'`) | Call `onMount(fn)` directly without import | `onMount` and `onDestroy` are compiler primitives automatically injected into the script scope. |
| Inline arrow function handlers (`on:click={() => doSomething()}`) | Named handler (`on:click={handleClick}`) | Wizz template syntax forbids inline function expressions in event directives. |
| Calling `persist()` inside nested blocks or functions | Initialize only as top-level `let x = persist(...)` | Compiler requires top-level binding to generate reactive storage getters and cross-tab sync listeners. |
| Manual `window.addEventListener('storage')` for tab sync | Use `persist(key, default)` | Wizz's built-in persistence already synchronizes storage changes across tabs reactively. |
| Modifying state in `onDestroy` | Perform cleanup only (clear timers, abort signals) | The component is being torn down; state mutations during destroy waste CPU cycles and trigger no render. |
