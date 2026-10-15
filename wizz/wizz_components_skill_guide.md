# Wizz Native — Component Architecture & Composition

## Component Architecture Decision Matrix

| Goal | Tool / Construct | Why |
| :--- | :--- | :--- |
| Component file definition | `.wizz` file with single native root | Mandatory file contract; parser enforces exactly one native root element per template. |
| Inject head metadata | `<wizz:head>` block | Direct, zero-runtime metadata injection for `<title>`, `<meta>`, and `<link>` elements. |
| Component-scoped styling | `<wizz:style>` block | Scoped by compiler via `[data-wizz-s]` attributes and auto-rewritten keyframes; extracted to `dist/app.css`. |
| Pass data to child component | Attribute props (`<Counter count={val} />`) | Declarative prop interface; props are reactive and strictly read-only in the child component. |
| Text interpolation | Safe expression `{expr}` | Escapes `&`, `<`, and `>`; limited to identifiers, member access, literals, and strict equality. |
| Reusable component inclusion | Default import (`import Comp from './Comp.wizz'`) | Static compile-time resolution; must be rendered self-closing inside a native parent. |

---

## File Structure & Root Element Contract

Every `.wizz` file combines markup, logic, scoped styling, and metadata in a single file. The template **must contain exactly one native root element**. Top-level siblings are strictly forbidden.

### Canonical Component Structure

```html
<!-- src/components/UserProfile.wizz -->
<div class="user-card">
  <header class="header">
    <h2 class="username">{user.name}</h2>
    <span class="role">{user.role}</span>
  </header>

  <p class="status-bio">{user.bio}</p>

  <footer class="actions">
    <button type="button" on:click={handleContact}>Contact</button>
  </footer>
</div>

<script>
  export let user;

  function handleContact() {
    console.log("Contacting", user.name);
  }
</script>

<wizz:head>
  <title>{user.name} - Profile</title>
  <meta name="description" content="User profile details" />
</wizz:head>

<wizz:style>
  .user-card {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    padding: 1.5rem;
    border: 1px solid #232d3a;
    border-radius: 8px;
  }

  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .username {
    margin: 0;
    font-size: 1.25rem;
  }
</wizz:style>
```

---

## Safe Template Expressions

Template curly braces (`{...}`) do **not** accept arbitrary JavaScript. The Wizz compiler restricts expressions to a safe, deterministic subset to eliminate runtime evaluation overhead:

- **Allowed**:
  - Identifiers (`{count}`)
  - Member access chains (`{user.address.city}`)
  - Number and string literals (`{100}`, `{"active"}`)
  - Strict equality comparisons (`{status === "ready"}`)
- **Forbidden**:
  - Function calls (`{formatDate(timestamp)}` $\rightarrow$ precompute in `<script>`)
  - Arithmetic operations (`{a + b}` $\rightarrow$ precalculate in `<script>`)
  - Ternary operators (`{isActive ? "on" : "off"}` $\rightarrow$ use `{#if}` or precalculate)
  - Method execution (`{items.map(...)}` $\rightarrow$ use `{#each}`)

### Escaping Rules
- Text interpolation automatically escapes HTML special characters: `&`, `<`, `>`.
- Attribute interpolation (`<div class="{className}">`) additionally escapes double quotes `"`.

---

## Component Composition & Child Rules

Wizz enforces strict composition boundaries:

1. **Default Imports Only**:
   ```javascript
   import Counter from './Counter.wizz';
   ```
2. **Self-Closing Inside Native Elements**:
   Child components must be rendered as self-closing tags and must be enclosed within a native DOM element:
   ```html
   <div class="container">
     <Counter count={initialCount} />
   </div>
   ```
3. **No Slot / Children Tags**:
   Imported component tags do not accept children:
   ```html
   <!-- INVALID / COMPILE ERROR -->
   <Counter><span slot="icon">+</span></Counter>
   ```
4. **No Direct Event Directives on Component Tags**:
   Event directives (`on:<event>`) cannot be attached directly to imported component tags. Pass callbacks as props or listen via event delegation:
   ```html
   <!-- INVALID / COMPILE ERROR -->
   <Counter on:change={handleChange} />

   <!-- VALID: Pass callback function prop -->
   <Counter onChange={handleChange} />
   ```

---

## `<wizz:head>` Metadata Management

Use `<wizz:head>` to declare document-level metadata without external libraries or client-side DOM queries:

- Exactly **one** `<wizz:head>` block per `.wizz` file.
- Permitted children: `<title>`, `<meta>`, and `<link>` only.
- Rendered during server-side rendering (SSR) and injected into the document shell's `<head>`.

```html
<wizz:head>
  <title>Dashboard | Wizz App</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <link rel="icon" href="/favicon.svg" type="image/svg+xml" />
</wizz:head>
```

---

## Scoped Styles with `<wizz:style>`

Wizz compiles `<wizz:style>` blocks into static, zero-runtime CSS with zero bundle bloat:

- **Attribute Hashing**: Scopes selectors with a compiler-generated attribute selector: `[data-wizz-s="<hash>"]`.
- **Keyframe Isolation**: Keyframes declared inside `<wizz:style>` are automatically rewritten with component hashes to eliminate global animation name collisions.
- **Single Block Rule**: At most one `<wizz:style>` block per `.wizz` file.
- **Global Styles Separation**: Place global reset rules and design tokens (`:root`) in `App.css`, never inside component `<wizz:style>` blocks.

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| Multiple root elements in template (`<div></div><div></div>`) | Single native wrapper element (`<div class="wrap">...</div>`) | Compiler diagnostic WIZZ-P001; parser strictly requires exactly one root element. |
| Complex inline expressions (`{items.length > 0 ? "yes" : "no"}`) | Precomputed reactive variable in `<script>` or `{#if}` | Wizz template parser does not evaluate arbitrary JS; fails at compile time. |
| Function calls in templates (`{calculateTotal(cart)}`) | Precompute `let total` in `<script>` | Template expressions only allow identifiers, member access, literals, and strict equality. |
| Children inside component tags (`<Button>Click me</Button>`) | Pass label as prop (`<Button label="Click me" />`) | Wizz component tags do not support children or slots. |
| Direct event directives on components (`<Modal on:close={fn} />`) | Pass handler prop (`<Modal onClose={fn} />`) | `on:<event>` directives are restricted to native DOM elements. |
| Multiple `<wizz:style>` or `<wizz:head>` blocks | Exactly one `<wizz:style>` and one `<wizz:head>` per file | Multiple blocks violate component contract and fail compilation. |
| Non-metadata elements in `<wizz:head>` (`<wizz:head><script>...</script></wizz:head>`) | Only `<title>`, `<meta>`, and `<link>` | Parser restricts head block children for security and deterministic SSR hydration. |
