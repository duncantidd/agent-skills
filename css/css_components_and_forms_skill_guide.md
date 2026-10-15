# CSS Native — Modern UI Components & Form Controls

## Components Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Accessible modals & dialogs | `<dialog>` + `::backdrop` | Renders in the browser top-layer with automatic focus trapping and ESC-key dismiss. |
| Zero-JS dropdowns & tooltips | Popover API (`popover`) | Native light-dismiss, top-layer stacking, and focus management without external libraries. |
| Exclusive group accordions | `<details name="...">` | Opening one accordion item automatically closes sibling items without JavaScript. |
| Auto-expanding text areas | `field-sizing: content` | Form inputs expand naturally with user text; completely eliminates JS resize listeners. |
| Tinting native form controls | `accent-color` | Quickly brands native checkboxes, radios, range sliders, and progress bars. |
| Custom smooth accordion animations | `interpolate-size: allow-keywords` | Enables smooth transitions to `height: auto` on expanding elements. |

---

## Native `<dialog>` Modals

The HTML `<dialog>` element renders into the browser's native **top layer**, meaning it will never be clipped by `overflow: hidden` or trapped by `z-index` wars.

```css
dialog {
  /* Reset standard browser border and styling */
  border: none;
  border-radius: 12px;
  padding: 2rem;
  background-color: var(--bg-surface);
  color: var(--text-main);
  box-shadow: 0 20px 25px -5px oklch(0 0 0 / 0.15);
  max-inline-size: min(90vw, 540px);
  margin: auto; /* Centers in viewport cleanly */

  /* Enter/Exit transition */
  opacity: 0;
  transform: scale(0.95);
  transition: opacity 200ms ease, transform 200ms ease,
              display 200ms allow-discrete, overlay 200ms allow-discrete;
}

/* Open state when activated via dialog.showModal() */
dialog[open] {
  opacity: 1;
  transform: scale(1);

  @starting-style {
    opacity: 0;
    transform: scale(0.95);
  }
}

/* Modal backdrop styling */
dialog::backdrop {
  background-color: oklch(0 0 0 / 0.5);
  backdrop-filter: blur(4px);
  opacity: 0;
  transition: opacity 200ms ease, display 200ms allow-discrete, overlay 200ms allow-discrete;
}

dialog[open]::backdrop {
  opacity: 1;

  @starting-style {
    opacity: 0;
  }
}
```

---

## The Popover API (`[popover]`)

For menus, context tooltips, and action sheets that don't need a modal backdrop:

```html
<!-- HTML Trigger and Popover -->
<button popovertarget="user-menu">My Profile</button>

<div id="user-menu" popover>
  <nav>
    <a href="/settings">Settings</a>
    <a href="/logout">Log Out</a>
  </nav>
</div>
```

```css
/* Popover element styling */
[popover] {
  border: 1px solid var(--border-subtle);
  border-radius: 8px;
  padding: 0.5rem;
  background-color: var(--bg-surface);
  box-shadow: 0 10px 15px -3px oklch(0 0 0 / 0.1);
  margin: 0;
  
  /* Smooth discrete enter/exit */
  opacity: 0;
  transform: translateY(-8px);
  transition: opacity 150ms ease, transform 150ms ease,
              display 150ms allow-discrete, overlay 150ms allow-discrete;
}

[popover]:popover-open {
  opacity: 1;
  transform: translateY(0);

  @starting-style {
    opacity: 0;
    transform: translateY(-8px);
  }
}
```

---

## Exclusive Group Accordions (`<details name="...">`)

Group multiple `<details>` elements together using the standard `name` attribute so that only one section remains open at a time:

```html
<details name="faq-group" open>
  <summary>What is this skill repository?</summary>
  <p>A collection of modern, production-grade instructions for AI agents.</p>
</details>

<details name="faq-group">
  <summary>Does it require JavaScript?</summary>
  <p>No, standard features solve these problems natively.</p>
</details>
```

```css
details {
  border-bottom: 1px solid var(--border-subtle);
  padding-block: 1rem;
}

summary {
  font-weight: 600;
  cursor: pointer;
  list-style: none; /* Removes default triangle marker */
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* Custom animated chevron */
summary::after {
  content: "›";
  font-size: 1.4rem;
  transition: transform 200ms ease;
}

details[open] summary::after {
  transform: rotate(90deg);
}

/* Enable smooth transitions to auto height */
:root {
  interpolate-size: allow-keywords;
}

details::details-content {
  opacity: 0;
  block-size: 0;
  overflow: clip;
  transition: content-visibility 300ms allow-discrete,
              opacity 300ms ease,
              block-size 300ms ease;
}

details[open]::details-content {
  opacity: 1;
  block-size: auto;
}
```

---

## Modern Form Controls

### 1. Intrinsic Auto-Sizing (`field-sizing: content`)
Inputs and textareas expand to fit text as the user types:

```css
textarea.auto-grow {
  field-sizing: content;
  min-block-size: 4lh;  /* Minimum 4 lines */
  max-block-size: 12lh; /* Maximum 12 lines before scrolling */
  resize: none;         /* User drag handle is unnecessary */
}
```

### 2. Accent Color
Brand native interactive controls with one line:

```css
:root {
  accent-color: var(--brand-accent);
}

/* Checkboxes, radio buttons, and range sliders now automatically adopt --brand-accent */
input[type="checkbox"],
input[type="radio"],
input[type="range"],
progress {
  accent-color: oklch(0.6 0.22 250);
}
```

### 3. Modern Checkbox & Radio Alignment
Align labels and inputs reliably without fragile `margin-top` nudges:

```css
.checkbox-label {
  display: inline-flex;
  align-items: baseline;
  gap: 0.5rem;
  cursor: pointer;
}

.checkbox-label input[type="checkbox"] {
  /* Aligns center of checkbox with text cap-height */
  transform: translateY(0.1em);
}
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| Heavy third-party modal libraries (React Modal, Bootstrap Modal) | Native `<dialog>` | Native dialog provides automatic top-layer rendering, focus trapping, and accessibility for free. |
| JavaScript click handlers to toggle dropdowns | HTML Popover API (`popover` attribute) | Popover API handles light dismiss (clicking outside) and keyboard ESC natively. |
| JavaScript accordion loops to close siblings | `<details name="group-name">` | Native browser exclusive grouping avoids state desynchronization and event listener overhead. |
| JS height measuring scripts for auto-growing textareas | `field-sizing: content` | Eliminates expensive `scrollHeight` layout recalculations on every keystroke. |
| Rebuilding custom radio/checkbox from scratch with hidden inputs | Native inputs with `accent-color` or `:has()` styling | Custom pseudo-element hacks often break screen reader focus and keyboard spacebar selection. |
| Modal dialogs without `::backdrop` styling | Style `dialog::backdrop` | Unstyled backdrops don't visually obscure page content, causing cognitive overload. |
