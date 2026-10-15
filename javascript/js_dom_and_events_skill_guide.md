# JavaScript Native — Modern DOM & Event Architecture

## DOM & Events Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Bulk event listener teardown | `addEventListener(..., { signal })` | Removes listeners automatically when `controller.abort()` is called; no need to track function references. |
| Empty and replace element children | `element.replaceChildren(...nodes)` | Fast, native, atomic replacement without `innerHTML = ''` or child-removal while loops. |
| Safe HTML generation | `<template>` or `DocumentFragment` | Prevents XSS script execution; parses markup offline before inserting into live DOM. |
| Viewport entry / Infinite scroll | `IntersectionObserver` | Efficient compositor-level tracking; avoids expensive `scroll` event listeners and layout thrashing. |
| Element size change detection | `ResizeObserver` | Observes individual element box changes without polling `window.onresize`. |
| Event delegation for dynamic lists | `event.target.closest(selector)` | Attaches a single listener on parent that matches any current or future child elements. |
| Form serialization to object | `Object.fromEntries(new FormData(form))` | Reads all inputs, checkboxes, and textareas into a clean key-value object in one line. |

---

## Declarative Event Cleanup with `AbortSignal`

The traditional `removeEventListener` requires storing an exact reference to the original handler function. If the handler was an anonymous arrow function, or if context changed, cleanup silently fails and leaks memory.

Using `{ signal }` lets you tear down hundreds of listeners in a single line:

```javascript
class Component {
  #controller = new AbortController();

  mount() {
    const { signal } = this.#controller;

    // All listeners share the same signal
    window.addEventListener('resize', this.#onResize, { signal });
    window.addEventListener('keydown', this.#onKey, { signal });
    document.addEventListener('click', this.#onClick, { signal });
  }

  unmount() {
    // Instantly tears down ALL listeners registered with this signal!
    this.#controller.abort();
  }

  #onResize = () => { /* ... */ };
  #onKey = () => { /* ... */ };
  #onClick = () => { /* ... */ };
}
```

---

## Safe DOM Mutation (`replaceChildren` & Safe Insertion)

### 1. Atomic Child Replacement
Never clear an element using `element.innerHTML = ''`:

```javascript
const list = document.getElementById('user-list');

const newItems = users.map(user => {
  const li = document.createElement('li');
  li.textContent = user.name; // Safe against XSS
  return li;
});

// Empties all existing children and appends newItems in one reflow
list.replaceChildren(...newItems);
```

### 2. Template Cloning for Complex Markup
Avoid concatenating strings into `innerHTML`. Use HTML `<template>` tags:

```html
<template id="card-template">
  <div class="card">
    <h3 class="title"></h3>
    <p class="body"></p>
  </div>
</template>
```

```javascript
const template = document.getElementById('card-template');
const clone = template.content.cloneNode(true);

// Populate text safely via textContent (immune to injection)
clone.querySelector('.title').textContent = untrustedUserData.title;
clone.querySelector('.body').textContent = untrustedUserData.body;

document.getElementById('feed').append(clone);
```

---

## Event Delegation with `.closest()`

Instead of attaching click listeners to 500 table rows or list items, attach a single listener on the parent container:

```javascript
const table = document.querySelector('table');

table.addEventListener('click', (event) => {
  // Finds the nearest button with [data-action], even if user clicked an icon inside it
  const actionBtn = event.target.closest('button[data-action]');
  if (!actionBtn || !table.contains(actionBtn)) return;

  const action = actionBtn.dataset.action;
  const rowId = actionBtn.closest('tr')?.dataset.id;
  
  handleTableAction(action, rowId);
});
```

---

## Observers: Eliminating Scroll & Resize Jank

### 1. `IntersectionObserver` (Lazy loading & reveal animations)
```javascript
const observer = new IntersectionObserver((entries, obs) => {
  for (const entry of entries) {
    if (entry.isIntersecting) {
      entry.target.classList.add('visible');
      obs.unobserve(entry.target); // Run once
    }
  }
}, { threshold: 0.1 });

document.querySelectorAll('.animate-on-scroll').forEach(el => observer.observe(el));
```

### 2. `ResizeObserver` (Responsive widget sizing)
```javascript
const chartObserver = new ResizeObserver((entries) => {
  for (const entry of entries) {
    const { width, height } = entry.contentRect;
    redrawChart(width, height);
  }
});

chartObserver.observe(document.getElementById('analytics-chart'));
```

---

## Native Form Serialization

Do not manually query each input element:

```javascript
const form = document.querySelector('form');

form.addEventListener('submit', (e) => {
  e.preventDefault();

  const formData = new FormData(form);
  
  // Converts all named inputs into a standard JavaScript object
  const payload = Object.fromEntries(formData);
  console.log('Form data:', payload);
});
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| `element.innerHTML = untrustedInput` | `element.textContent` or `<template>` | High-risk XSS vulnerability; executes injected `<script>` or `onerror` handlers. |
| `while (el.firstChild) el.removeChild(el.firstChild)` | `element.replaceChildren()` | Native `replaceChildren` is atomic, faster, and triggers only one DOM mutation event. |
| Manual `removeEventListener` with separate references | `{ signal: controller.signal }` | Eliminates memory leaks and dangling listeners when component destroys. |
| `window.addEventListener('scroll', handler)` for reveals | `IntersectionObserver` | Scroll handlers run synchronously on the main thread and cause severe scroll stuttering. |
| Polling `window.onresize` for card layouts | `ResizeObserver` or CSS Container Queries | Adapts directly to the element's actual container dimensions rather than whole window. |
| Querying inputs one by one: `form.querySelector('#name')` | `new FormData(form)` | Misses dynamic inputs, disabled states, and checkboxes; `FormData` is standard and complete. |
