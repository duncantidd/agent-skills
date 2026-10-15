# Wizz Native — Template Directives & Control Flow

## Directives Decision Matrix

| Goal | Tool / Directive | Why |
| :--- | :--- | :--- |
| Conditional rendering | `{#if condition} ... {:else} ... {/if}` | Compile-time evaluated branching evaluated at component mount. |
| Keyed list iteration | `{#each items as item (item.id)}` | Stable reconciliation when list elements are sorted, prepended, or removed. |
| Positional list iteration | `{#each items as item}` | Keyless iteration for static or append-only lists where item identity is positional. |
| Item interactions in a list | Event delegation on container + `event.target.closest()` | `{#each}` bodies forbid `on:` directives; container event delegation is native, fast, and idiomatic. |
| Complex child layout in list | Native element with semantic child elements | `{#each}` bodies forbid imported components; render native markup inside each item. |

---

## Conditional Blocks (`{#if}`)

The `{#if}` directive provides structural branching in templates:

- Condition expressions use the safe expression subset (e.g. identifier `{isLoggedIn}` or comparison `{status === "ready"}`).
- The optional `{:else}` branch provides the alternative markup.
- Branches are chosen at mount time.

```html
<!-- src/components/AuthStatus.wizz -->
<div class="auth-bar">
  {#if isAuthenticated}
    <div class="user-greeting">
      <span>Welcome back, {userName}!</span>
    </div>
  {:else}
    <div class="login-prompt">
      <span>Please log in to continue.</span>
    </div>
  {/if}
</div>

<script>
  export let isAuthenticated = false;
  export let userName = "Guest";
</script>
```

---

## List Iteration (`{#each}`) & Compiler Constraints

Wizz handles list rendering with a compiler-optimized loop. To ensure predictable performance, the compiler enforces **four strict structural constraints** on `{#each}` blocks:

1. **Native Parent Wrapper**: The `{#each}` block must be immediately enclosed within a native HTML element (`<ul>`, `<div>`, `<section>`). It cannot be the component root.
2. **Single Native Child**: The body of the `{#each}` loop must contain **exactly one native root element** (e.g. `<li>` or `<div>`).
3. **Reactive `let` Collection**: The collection being iterated must be declared as a reactive `let` in the `<script>` block.
4. **Restricted Loop Bodies**:
   - **No imported components** inside `{#each}` bodies.
   - **No `on:` event directives** inside `{#each}` bodies.
   - **No nested blocks** (`{#if}` or nested `{#each}`) inside `{#each}` bodies.

Violation of these rules triggers compile-time diagnostic errors (`WIZZ-G###` / `WIZZ-P###`).

### Canonical Keyed Iteration

```html
<!-- src/components/TodoList.wizz -->
<div class="todo-widget">
  <ul class="item-list">
    {#each todos as item (item.id)}
      <li class="todo-item" data-id="{item.id}">
        <span class="todo-text">{item.title}</span>
        <span class="todo-status">{item.status}</span>
      </li>
    {/each}
  </ul>
</div>

<script>
  let todos = [
    { id: "t1", title: "Review architectural contract", status: "done" },
    { id: "t2", title: "Compile Wizz templates", status: "active" },
    { id: "t3", title: "Deploy Node adapter", status: "pending" }
  ];
</script>
```

---

## Idiomatic Pattern: Event Delegation for List Items

Because `{#each}` blocks forbid direct `on:` directives inside their bodies, interactions on list items must be handled using **native event delegation** on the parent container element:

```html
<!-- src/components/InteractiveList.wizz -->
<div class="list-container">
  <ul class="task-list" on:click={handleItemClick}>
    {#each tasks as task (task.id)}
      <li class="task-row" data-id="{task.id}">
        <span class="task-title">{task.title}</span>
        <button type="button" class="delete-btn" data-action="delete">Remove</button>
      </li>
    {/each}
  </ul>
</div>

<script>
  let tasks = [
    { id: "1", title: "Write specification" },
    { id: "2", title: "Run static analyzer" }
  ];

  function handleItemClick(event) {
    const actionBtn = event.target.closest('[data-action="delete"]');
    if (!actionBtn) return;

    const row = actionBtn.closest('[data-id]');
    if (!row) return;

    const targetId = row.dataset.id;
    // Non-mutating filter reassignment triggers reactive DOM update
    tasks = tasks.filter(t => t.id !== targetId);
  }
</script>
```

---

## Idiomatic Pattern: Precomputing Data for Conditionals in Lists

Because nested `{#if}` blocks are forbidden inside `{#each}`, transform or annotate data items in `<script>` before binding to the template:

```html
<!-- src/components/BadgeList.wizz -->
<div class="badges-wrap">
  <div class="grid">
    {#each formattedItems as item (item.id)}
      <article class="badge-card {item.badgeClass}">
        <h3>{item.title}</h3>
        <span class="indicator">{item.displayStatus}</span>
      </article>
    {/each}
  </div>
</div>

<script>
  let rawItems = [
    { id: "a", title: "Server Health", isHealthy: true },
    { id: "b", title: "Storage Space", isHealthy: false }
  ];

  // Precompute presentation attributes rather than nesting {#if} in {#each}
  let formattedItems = rawItems.map(item => ({
    id: item.id,
    title: item.title,
    badgeClass: item.isHealthy ? "status-ok" : "status-warning",
    displayStatus: item.isHealthy ? "ONLINE" : "ATTENTION"
  }));
</script>
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| Placing components inside `{#each}` (`{#each items as item} <Card item={item} /> {/each}`) | Render native elements in `{#each}` (`{#each items as item} <div class="card">...</div> {/each}`) | Wizz compiler strictly forbids imported components in `{#each}` bodies. |
| Attaching `on:<event>` inside `{#each}` (`<button on:click={fn}>`) | Use event delegation on parent container (`<ul on:click={handleListClick}>`) | `on:` directives inside `{#each}` are rejected by the compiler. |
| Nesting `{#if}` inside `{#each}` | Precompute display flags or CSS classes in `<script>` | Nested template blocks inside `{#each}` trigger compile-time errors. |
| Floating `{#each}` without native wrapper | Wrap in `<div class="container">` or `<ul>` | `{#each}` blocks must reside within a native DOM parent element. |
| Multiple child elements inside `{#each}` | Exactly one native element per `{#each}` iteration | The loop body must contain a single native root element. |
| Iterating non-reactive variables in `{#each}` | Declare source collections with `let items = [...]` | Wizz reactivity binds only to top-level `let` declarations. |
| Mutating arrays with `.push()` or `.splice()` | Reassign array (`items = [...items, newItem]`) | Reassignment triggers the microtask reactivity scheduler; in-place array mutation does not. |
