# CSS Native — Architecture & Scoping

## Architecture Decision Matrix

| Goal | Tool | Why | 
| ----- | ----- | ----- | 
| Managing specificity wars | Cascade Layers (`@layer`) | Defines a strict hierarchy where higher layers always win, regardless of selector specificity. | 
| Resilient overrides & defaults | `:where()` pseudo-class | Applies styles with exactly **0 specificity**, allowing any subsequent selector to override effortlessly. | 
| Grouping selectors without specificity bloat | `:is()` pseudo-class | Consolidates complex selector lists; adopts the specificity of its highest-specificity argument. | 
| Component-specific styling | `@scope` | Prevents styles from bleeding out, and prevents global styles from bleeding in (donut scoping). | 
| Writing cleaner, hierarchical CSS | Native Nesting | Eliminates the need for preprocessors (Sass/Less) to group related selectors. | 
| Baseline consistent layout | Modern Minimal Reset | Sets modern box-sizing, fluid media, and clean typography defaults without legacy reset bloat. | 
| Theming (Dark/Light mode) | `light-dark()` | Toggles color values automatically based on the `color-scheme` property without media queries. | 
| Reusable design tokens | Custom Properties (`--var`) & `@property` | Defines typed, single-source-of-truth variables at the `:root` level with fallback defaults. | 

## Cascade Layers (`@layer`)

Layers fix the "specificity wars" by explicitly declaring which styles take precedence. A style in a higher layer will *always* beat a style in a lower layer, even if the lower layer has a higher selector specificity (like an ID).

```css
/* 1. Define the order of layers upfront (lowest to highest priority) */
@layer reset, base, theme, components, utilities;

/* 2. Assign styles to layers */
@layer reset {
  /* Low priority */
  h1 { margin: 0; }
}

@layer utilities {
  /* High priority */
  .mt-0 { margin-block-start: 0; }
}

@layer components {
  /* Even though an ID (#hero) has high specificity, 
     it loses to .mt-0 because 'utilities' is a higher layer. */
  #hero h1 { margin-block-start: 2rem; }
}
```

*Tip: Unlayered styles (standard CSS written outside a `@layer` block) have the highest priority by default, beating all layered styles.*

## Native Scoping (`@scope`)

Scope limits the reach of your selectors, replacing complex naming conventions like BEM (`.card__title--large`). 

### Basic Scope
Target elements only inside a specific parent component.

```css
@scope (.card) {
  /* Only styles <img> tags inside .card */
  img {
    border-radius: 0.5rem;
  }
  
  /* The :scope pseudo-class refers to the .card itself */
  :scope {
    background: white;
  }
}
```

### "Donut" Scoping (Scope with lower boundaries)
Apply styles inside a component, but *stop* before reaching a nested component.

```css
/* Style everything inside .article, EXCEPT what is inside .ad-slot */
@scope (.article) to (.ad-slot) {
  p {
    color: var(--text-dark);
  }
}
```

## Native Nesting

You no longer need Sass or PostCSS for nesting. Modern CSS supports it natively.

```css
.button {
  background: blue;
  color: white;

  /* Pseudo-classes */
  &:hover {
    background: darkblue;
  }

  /* Structural nesting */
  .icon {
    margin-inline-end: 0.5rem;
  }

  /* Media queries can be nested directly inside the selector */
  @media (max-width: 600px) {
    width: 100%;
  }
}
```

## Native Light/Dark Theming

The `light-dark()` function simplifies theming by tying colors directly to the `color-scheme` property.

```css
:root {
  /* Tells the browser this page supports both modes */
  color-scheme: light dark;
  
  /* light-dark(light-value, dark-value) */
  --bg-surface: light-dark(#ffffff, #121212);
  --text-primary: light-dark(#111111, #eeeeee);
}

body {
  background-color: var(--bg-surface);
  color: var(--text-primary);
}

/* Force dark mode on a specific component, regardless of OS preference */
.dark-card {
  color-scheme: dark;
}
```

---

## Specificity Management: `:is()` vs `:where()`

Both `:is()` and `:where()` accept a list of selectors, but handle specificity completely differently:

*   **`:is(...)`**: Adopts the specificity of its **highest** selector argument. Useful for condensing selectors.
*   **`:where(...)`**: Has **0 specificity** regardless of what's inside. Ideal for baseline styles and design systems that callers must easily override.

```css
/* Zero-specificity baseline defaults: easily overridden by ANY class or id later */
:where(h1, h2, h3, h4) {
  line-height: 1.2;
  text-wrap: balance;
  margin-block: 0 0.5em;
}

/* Later in your code, even an unlayered single class wins cleanly: */
.custom-title {
  line-height: 1.4; /* Overrides :where(...) with zero struggle */
}

/* Condensing repetitive selectors with :is() */
:is(header, main, footer) a:hover {
  text-decoration: underline;
}
```

---

## Modern Baseline CSS Reset

Every project benefits from an unbloated modern baseline that establishes predictable geometry, typography inheritance, and responsive media:

```css
@layer reset {
  /* 1. Use a more-intuitive box-sizing model */
  *, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
  }

  /* 2. Responsive media and canvas by default */
  img, picture, video, canvas, svg {
    display: block;
    max-inline-size: 100%;
  }

  /* 3. Form controls inherit font properties */
  input, button, textarea, select {
    font: inherit;
    color: inherit;
  }

  /* 4. Text wrap defaults */
  p, h1, h2, h3, h4, h5, h6 {
    overflow-wrap: break-word;
  }

  /* 5. Root page flow */
  body {
    min-block-size: 100dvh;
    line-height: 1.5;
    -webkit-font-smoothing: antialiased;
  }
}
```

---

## Robust Custom Properties & Typed Design Tokens

### Fallbacks for Safety
Always provide resilient fallbacks when consuming variables in reusable UI components:

```css
.badge {
  /* Uses --badge-bg if provided; otherwise falls back to system canvas */
  background-color: var(--badge-bg, light-dark(#e2e8f0, #334155));
  color: var(--badge-fg, currentColor);
}
```

### Typed Tokens with `@property`
Declare variable types to prevent invalid assignments and enable transition/animation:

```css
@property --border-thickness {
  syntax: "<length>";
  inherits: true;
  initial-value: 1px;
}
```

---

## Do Not

| BAD | GOOD | Why | 
| ----- | ----- | ----- | 
| `!important` | `@layer` or `:where()` | `!important` breaks cascade order and creates specificity dead-ends. | 
| `.card .card-body .card-title span` | `.card-title` | Deep nesting creates brittle selectors and balloons stylesheet weight. | 
| Hardcoded hex values in components | CSS Custom Properties (`var(--color)`) | Centralizes design tokens in `:root` for effortless rebranding and theming. | 
| Heavy legacy reset stylesheets (Normalize.css) | Minimal Modern Reset via `@layer reset` | Modern browsers share consistent baselines; massive 500-line resets add dead weight. | 
| High-specificity resets (`html body div h1`) | `:where(h1, h2, h3)` | `:where()` has 0 specificity, ensuring user-defined component classes always win. | 
| `.button--primary__icon` (Strict BEM) | `@scope` | BEM was invented to solve global CSS leakage; `@scope` solves this natively. | 
| Over-using `*` selector in descendents | Targeting specific components | Deep universal rules (`.card *`) force layout and paint recalculation on every element. | 
