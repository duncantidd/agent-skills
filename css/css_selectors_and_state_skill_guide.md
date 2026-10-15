# CSS Native — Modern Selectors & State Management

## Selectors Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Style parent based on child or sibling | `:has()` | The "relational selector"; eliminates JS class toggling for parent state changes. |
| User-friendly form validation | `:user-valid` / `:user-invalid` | Only highlights invalid fields *after* the user interacts, unlike `:invalid` which errors immediately. |
| Keyboard-only focus rings | `:focus-visible` | Shows focus indicators for keyboard navigation while suppressing them for mouse clicks. |
| Container focus awareness | `:focus-within` | Highlights an entire input card, row, or modal when any child element receives focus. |
| Zero-specificity baseline rules | `:where()` | Groups selectors with 0 specificity to ensure effortless overrides by consumer components. |
| Grouping without repeating selectors | `:is()` | Consolidates complex selector lists into a clean, DRY format. |
| Floating label without JavaScript | `:placeholder-shown` | Detects whether an input field has entered content or is showing placeholder text. |
| Native modal / popover state | `:popover-open` / `dialog[open]` | Styles native top-layer elements when open without synthetic JS open classes. |

---

## The `:has()` Relational Selector

`:has()` is the most powerful addition to CSS selectors, enabling parent, ancestor, and previous-sibling styling without JavaScript event listeners.

### 1. Form Validation Container Styling
Highlight the whole field container when the internal input is invalid after interaction:

```css
.form-group:has(:user-invalid) {
  border-color: oklch(0.6 0.22 25);
  background-color: oklch(0.6 0.22 25 / 0.05);
}

.form-group .error-message {
  display: none;
}

.form-group:has(:user-invalid) .error-message {
  display: block;
}
```

### 2. Layout Conditioning via Child Presence
Adapt card or list layouts based on media presence:

```css
/* Card with an image has a 2-column layout; card without image stays 1-column */
.card:has(> img) {
  display: grid;
  grid-template-columns: 140px 1fr;
  gap: 1.5rem;
}
```

### 3. Quantity Queries
Style elements based on how many items exist in a list:

```css
/* If list has 5 or more children, render in a tighter compact grid */
.tags-list:has(> :nth-child(5)) {
  grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
  font-size: 0.85rem;
}
```

### 4. Interactive Sibling Coordination
Dim surrounding items when hovering one item in a gallery:

```css
.gallery:has(.item:hover) .item:not(:hover) {
  opacity: 0.5;
  filter: grayscale(40%);
  transition: opacity 200ms ease, filter 200ms ease;
}
```

---

## Ergonomic Form States: `:user-valid` vs `:invalid`

The legacy `:invalid` pseudo-class triggers as soon as the page loads, presenting intimidating red borders before the user has even clicked into a required field.

Use `:user-valid` and `:user-invalid` to trigger feedback only after user interaction:

```css
input {
  border: 1px solid var(--border-color);
  outline: none;
}

/* Shows green ONLY after the user typed valid input and blurred, or submitted */
input:user-valid {
  border-color: oklch(0.65 0.18 145);
}

/* Shows red ONLY after the user typed invalid data or failed validation on submit */
input:user-invalid {
  border-color: oklch(0.6 0.22 25);
}
```

---

## Focus Management: `:focus-visible` vs `:focus-within`

Never remove focus rings with `outline: none` without providing an accessible alternative.

```css
/* 1. Global Focus Indicator for keyboard navigators */
:focus-visible {
  outline: 2px solid var(--brand-accent);
  outline-offset: 2px;
}

/* Suppress ugly outline for mouse/touch clicks while keeping it for keyboard Tab */
button:focus:not(:focus-visible) {
  outline: none;
}

/* 2. Container Awareness with :focus-within */
.search-bar {
  border: 1px solid var(--border-subtle);
  transition: border-color 150ms ease, box-shadow 150ms ease;
}

/* Highlights the entire search bar container when input OR button inside is focused */
.search-bar:focus-within {
  border-color: var(--brand-accent);
  box-shadow: 0 0 0 3px oklch(from var(--brand-accent) l c h / 0.2);
}
```

---

## Pure CSS Floating Labels (`:placeholder-shown`)

Create smooth floating input labels with zero JavaScript:

```css
.field-wrapper {
  position: relative;
}

.field-wrapper input {
  padding-block-start: 1.5rem;
}

.field-wrapper label {
  position: absolute;
  inset-block-start: 1rem;
  inset-inline-start: 0.75rem;
  transition: transform 150ms ease, font-size 150ms ease, color 150ms ease;
  pointer-events: none;
}

/* When input has text (placeholder is hidden) OR has focus, float label up */
.field-wrapper input:focus + label,
.field-wrapper input:not(:placeholder-shown) + label {
  transform: translateY(-0.6rem);
  font-size: 0.75rem;
  color: var(--brand-accent);
}
```

---

## Native Top-Layer States

Style native modals and popovers based on their HTML5 standard open states:

```css
/* Dialog element when active */
dialog[open] {
  animation: dialog-fade-in 200ms ease-out;
}

/* Popover API target when toggled visible */
[popover]:popover-open {
  animation: popover-drop 150ms ease-out;
}
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| `input:invalid` for initial form styling | `input:user-invalid` | `:invalid` shows error borders on empty required fields immediately upon page load. |
| `outline: none` on interactive controls | `:focus-visible { outline: 2px solid ... }` | Disabling focus rings renders pages completely unusable for keyboard navigators. |
| JavaScript event listeners to toggle parent classes | `:has()` relational selector | `:has()` eliminates DOM event dispatching, reflow loops, and brittle sync logic. |
| `a:hover, button:hover, input:hover` repeating complex selectors | `:is(a, button, input):hover` | `:is()` reduces selector duplication and keeps stylesheets concise. |
| Heavy utility class specificity like `.table.table-striped td` | `:where(.table) td` | `:where()` has 0 specificity, allowing consumers to override table cells without specificity hacks. |
| Using `target="_blank"` without visual or programmatic indicators | `a[target="_blank"]::after { content: " ↗"; }` | Informs users that a link will open in a new window before they activate it. |
