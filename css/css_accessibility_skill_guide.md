# CSS Native — Accessibility & Preference Ergonomics

## Accessibility Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Accessible content for screen readers only | `.sr-only` (Visually Hidden) | Clips content visually while preserving it for assistive technology in the accessibility tree. |
| Vestibular disorder protection | `@media (prefers-reduced-motion)` | Disables or minimizes animations and transitions based on system accessibility settings. |
| Low vision & high contrast support | `@media (forced-colors)` / `System Colors` | Preserves interactive outlines and borders in Windows High Contrast mode. |
| Contrast preferences | `@media (prefers-contrast)` | Adjusts border weights and color contrast ratios based on user system preference. |
| Mobile touch accuracy | Touch Target Sizing (`min-size: 44px`) | Ensures interactive controls meet WCAG tap target requirements (minimum 24-44px). |
| Pointer-aware touch targets | `@media (pointer: coarse)` | Dynamically expands hit targets and spacing on touch devices without impacting desktop mouse users. |
| Text zoom resilience | Relative units (`rem` & `min-block-size`) | Allows pages to scale cleanly up to 200% zoom without clipping or overlapping text. |

---

## Screen Reader Only Utility (`.sr-only`)

Never use `display: none` or `visibility: hidden` for text intended for screen readers (e.g., icon-only buttons, skip links). Those properties remove the element from the accessibility tree entirely.

Use the modern zero-side-effect clipping pattern:

```css
.sr-only {
  position: absolute;
  inline-size: 1px;
  block-size: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
  border: 0;
}

/* Allow skip link to become visible on keyboard focus */
.sr-only-focusable:focus {
  position: fixed;
  inset-block-start: 1rem;
  inset-inline-start: 1rem;
  inline-size: auto;
  block-size: auto;
  padding: 0.75rem 1.25rem;
  margin: 0;
  overflow: visible;
  clip-path: none;
  white-space: normal;
  background-color: var(--bg-surface);
  color: var(--text-main);
  border: 2px solid var(--brand-accent);
  z-index: 10000;
}
```

---

## High-Visibility Keyboard Focus Rings

A keyboard user must never lose track of where they are on a page. Combine `:focus-visible` with `outline-offset` to ensure visibility over any background:

```css
:focus-visible {
  outline: 2px solid var(--brand-focus, Highlight);
  outline-offset: 3px;
  border-radius: 4px;
}

/* Double-ring technique: visible on both dark and light backgrounds */
.high-contrast-focus:focus-visible {
  outline: 2px solid black;
  box-shadow: 0 0 0 4px white;
}
```

---

## User Preference Media Queries

### 1. Motion Sensitivity (`prefers-reduced-motion`)
Respect users with vestibular disorders by disabling rapid or non-essential movement:

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### 2. High Contrast (`prefers-contrast: more`)
Increase border clarity and text contrast when requested by the OS:

```css
@media (prefers-contrast: more) {
  :root {
    --border-subtle: oklch(0.2 0 0);
    --text-muted: oklch(0.1 0 0);
  }
  
  .card, .button, input {
    border-width: 2px;
  }
}
```

### 3. Forced Colors Mode (Windows Contrast Themes)
In forced colors mode, colors are replaced with system keywords. Ensure custom SVG icons and borders adapt:

```css
@media (forced-colors: active) {
  .icon {
    /* Ensures SVGs adopt the system text color */
    fill: CanvasText;
  }

  .button {
    /* Guarantees button outline is visible even if flat styled */
    border: 2px solid ButtonText;
  }
}
```

---

## Pointer-Aware Tap Targets (`pointer: coarse`)

WCAG 2.2 Success Criterion 2.5.8 requires interactive targets to be at least 24x24px, with 44x44px recommended for mobile touch devices:

```css
.button, .nav-link, .icon-button {
  /* Default desktop hit target */
  min-inline-size: 36px;
  min-block-size: 36px;
}

/* Adapt automatically for touch screen users */
@media (pointer: coarse) {
  .button, .nav-link, .icon-button {
    min-inline-size: 44px;
    min-block-size: 44px;
  }
  
  /* Increase gap between clickable items to prevent accidental taps */
  .button-group {
    gap: 0.75rem;
  }
}
```

---

## Zoom & Font Scaling Resilience

Users with vision impairment scale text up to 200%. Hardcoded pixel heights cause text to spill out and overlap:

```css
/* BAD: Clamps container to 50px regardless of font scale */
.header-bad {
  height: 50px;
}

/* GOOD: Accommodates zoomed text gracefully */
.header-good {
  min-block-size: 3.5rem;
  padding-block: 0.75rem;
}
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| `display: none` on screen-reader text | `.sr-only` class with `clip-path` | `display: none` deletes the content from the accessibility tree, leaving blind users uninformed. |
| `outline: none` or `outline: 0` | `:focus-visible { outline: 2px solid ... }` | Completely breaks keyboard navigation, violating WCAG SC 2.4.7. |
| Hardcoded `px` on layout containers (`height: 300px`) | `min-block-size: 300px` | Clamped pixel heights cause text to overflow and clip when users zoom in. |
| Tiny icon buttons (`width: 20px; height: 20px`) | `min-inline-size: 44px; min-block-size: 44px;` | Causes mis-taps on mobile devices; fails WCAG tap target requirements. |
| `user-scalable=no` or `maximum-scale=1.0` in viewport meta | Let users zoom freely | Disabling zoom is a severe accessibility violation for low-vision users. |
| Relying solely on color to convey state (e.g. only red border for error) | Pair color with text or icons (`:has(:user-invalid) .error-msg`) | Color-blind users cannot reliably distinguish status from color alone (WCAG SC 1.4.1). |
| Overriding user reduced motion preferences | Honor `@media (prefers-reduced-motion: reduce)` | Vestibular triggers can induce motion sickness, nausea, and vertigo in users. |
