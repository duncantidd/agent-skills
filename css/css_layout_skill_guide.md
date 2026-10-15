# CSS Native — Modern Layout & Positioning

## Layout Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| 1D layout (row OR column) | Flexbox | Distributes space and aligns items along a single axis dynamically. |
| 2D layout (rows AND columns) | CSS Grid | Defines structured multi-track skeletons, alignments, and areas. |
| Aligning nested card contents | Subgrid | Allows child/grandchild elements to align to parent grid tracks. |
| Component-based responsiveness | Container Queries | Adapts component layout to parent container size, not viewport. |
| Centering element in container | `place-content: center` | One-liner centering on Grid or Flex without legacy positioning hacks. |
| Preventing Flex child overflow | `min-inline-size: 0` | Resets default `auto` min-size on flex items to permit text truncation. |
| Intrinsic auto-growing inputs | `field-sizing: content` | Lets textareas and inputs auto-expand without JavaScript resize listeners. |
| Layout variants via child presence | `:has()` relational selector | Conditionally styles parent layouts based on child count or components. |
| Text wrapping & balance | `text-wrap` | Use `balance` for headings, `pretty` to prevent orphans in paragraphs. |

---

## Modern CSS Grid Fundamentals

### The RAM Pattern (Repeat, Auto, Minmax)
The ultimate fluid grid without media queries.

```css
.auto-grid {
  display: grid;
  gap: 1.5rem;
  /* Auto-fit fills the space, minmax sets the floor (300px) and ceiling (1fr) */
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
}
```

### `auto-fit` vs `auto-fill`
- `auto-fit`: Collapses empty tracks. Items stretch to fill available space.
- `auto-fill`: Maintains empty tracks. Items maintain their `minmax` base size and leave gaps if there's extra room.

---

## Subgrid (Nested Alignment)

Solves the classic problem of aligning headers, bodies, and footers across multiple cards in a row, regardless of content length.

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
}

.card {
  display: grid;
  /* Span 3 rows in the parent grid (header, content, footer) */
  grid-template-rows: auto 1fr auto; 
}

/* If the parent grid had rows defined, subgrid allows the card to adopt them: */
.parent {
  display: grid;
  grid-template-rows: repeat(3, auto);
}
.child-card {
  display: grid;
  grid-template-rows: subgrid; /* Grandchildren now align across the parent's rows */
  grid-row: span 3;
}
```

---

## Logical Properties (Internationalization-Ready)

Stop using physical directions (`top`, `bottom`, `left`, `right`). Use flow-relative logical properties to ensure your layout automatically supports Right-to-Left (RTL) languages (like Arabic or Hebrew) and vertical writing modes.

### Dimensions
| Physical | Logical |
| :--- | :--- |
| `width` | `inline-size` |
| `height` | `block-size` |

### Spacing (Margin & Padding)
| Physical | Logical |
| :--- | :--- |
| `margin-top` / `margin-bottom` | `margin-block-start` / `margin-block-end` |
| `margin-left` / `margin-right` | `margin-inline-start` / `margin-inline-end` |
| `margin: 0 auto;` | `margin-inline: auto;` |
| `padding-top` / `padding-bottom`| `padding-block-start` / `padding-block-end` |

### Positioning
| Physical | Logical |
| :--- | :--- |
| `top: 0; bottom: 0; left: 0; right: 0;` | `inset: 0;` |
| `top: 1rem;` | `inset-block-start: 1rem;` |
| `left: 2rem;` | `inset-inline-start: 2rem;` |

---

## Modern Viewport Units

Mobile browser UI (URL bars, navigation footers) dynamically show and hide, causing older `100vh` layouts to jump or hide content.

*   **`dvh` (Dynamic Viewport Height):** Adapts as browser UI expands and retracts. Use this for full-screen hero sections.
*   **`svh` (Small Viewport Height):** The shortest possible viewport (browser UI fully visible). Great for sticky footers.
*   **`lvh` (Large Viewport Height):** The tallest possible viewport (browser UI hidden). 
*   **`dvw` (Dynamic Viewport Width):** Accounts for scrollbar widths on Windows, preventing horizontal overflow.

```css
.hero-section {
  min-block-size: 100dvh; /* Replaces min-height: 100vh */
  inline-size: 100%;
}
```

---

## Container Queries (Layout Shifting)

Breakpoints based on the size of a component's container, allowing a component to be dropped anywhere on the page and style itself correctly.

```css
.sidebar {
  container-type: inline-size;
  container-name: sidebar-area;
}

.user-profile {
  display: flex;
  flex-direction: column; /* Default: stacked */
}

/* If the container is wider than 400px, switch to a row */
@container sidebar-area (min-width: 400px) {
  .user-profile {
    flex-direction: row;
    align-items: center;
  }
}
```

---

## Flexbox Essentials & The Min-Size Overflow Trap

By default, flex items have `min-width: auto` (`min-inline-size: auto`), which causes children containing long text, URLs, or preformatted code to push out and break the layout instead of wrapping or truncating.

```css
.media-object {
  display: flex;
  gap: 1rem;
}

.media-content {
  /* CRITICAL: Allows flex item to shrink below its content's intrinsic size */
  min-inline-size: 0;
  flex: 1;
}

.media-content p {
  /* Now text-overflow: ellipsis or overflow-wrap actually works! */
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
```

---

## Modern Centering

Never use `top: 50%; left: 50%; transform: translate(-50%, -50%)`. Modern CSS centers elements cleanly:

### 1. Grid One-Liner (Best for container centering)
```css
.parent {
  display: grid;
  place-content: center; /* Centers along both axes */
  min-block-size: 100dvh;
}
```

### 2. Flex Auto Margins (Best for pushing items in flex containers)
```css
.parent {
  display: flex;
}
.child {
  margin: auto; /* Absorbs all leftover space in all directions */
}
```

---

## Intrinsic Form Sizing (`field-sizing: content`)

Inputs and textareas historically required JavaScript to grow with their content. CSS now natively provides auto-sizing form controls:

```css
textarea.auto-grow {
  field-sizing: content;
  min-block-size: 3lh;  /* At least 3 lines of text */
  max-block-size: 15lh; /* Caps height before scrolling */
}

input.auto-grow {
  field-sizing: content;
  min-inline-size: 12ch;
}
```

---

## Relational Layout with `:has()`

Style parent layouts or grid compositions conditionally without adding JavaScript state classes.

```css
/* Switch to a 3-column layout only when 3 or more children exist */
.gallery {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

.gallery:has(> :nth-child(3)) {
  grid-template-columns: repeat(3, 1fr);
}

/* Adapt main column when an aside sidebar is present */
.page-layout:has(aside) {
  display: grid;
  grid-template-columns: 280px 1fr;
}
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| `height: 100vh` | `min-block-size: 100dvh` | Prevents content clipping on mobile browsers with dynamic URL bars. |
| `width: 100vw` | `inline-size: 100%` | `100vw` includes the scrollbar track on Windows, triggering horizontal overflow. |
| `margin-left: 1rem` | `margin-inline-start: 1rem` | Physical properties break in RTL (Right-to-Left) languages. |
| Default flex items with long text | `min-inline-size: 0; flex: 1;` | Default `min-width: auto` prevents flex items from shrinking or truncating. |
| `@media (max-width: 768px)` for components | `@container (max-width: 500px)` | Media queries tightly couple components to viewport; containers keep them modular. |
| `top: 50%; transform: translateY(-50%)` | `display: grid; place-content: center;` | Positioning hacks require non-static positioning, break flow, and cause blur on subpixel displays. |
| Negative margins for grids | `gap: 1rem` | `gap` works natively in Flexbox and Grid without breaking container boundaries. |
| JavaScript for auto-growing textarea | `field-sizing: content` | CSS native auto-sizing avoids DOM polling, layout thrashing, and resize scripts. |
| Floats for multi-column layout | `display: flex` or `display: grid` | `float` is a legacy text-wrap tool that requires clearfix hacks and breaks document flow. |