# CSS Native — Colors, Spaces & Dynamic Theming

## Colors Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Perceptually uniform palette | OKLCH (`oklch()`) | Uniform lightness and chroma across all hues; eliminates unpredictable brightness jumps found in HSL. |
| Deriving tints, shades, opacities | Relative Color Syntax | Extracts L, C, H channels from an existing token to create variants without preprocessors or color variables. |
| Blending colors dynamically | `color-mix()` | Mixes two arbitrary colors in any color space directly at runtime. |
| Automatic Light/Dark mode | `light-dark()` | Seamlessly returns light or dark values tied to the element's `color-scheme`. |
| Windows High Contrast support | System Color Keywords | Preserves critical UI borders and accents in `forced-colors: active` mode. |
| Wide gamut displays (P3) | `display-p3` / `oklch()` | Accesses 30%+ more vibrant greens, pinks, and reds on modern screens. |

---

## OKLCH: The Modern Color Space

HSL is mathematically flawed: yellow at `lightness: 50%` appears blindingly bright, while blue at `50%` appears nearly black. 

**OKLCH** (Lightness, Chroma, Hue) solves this by matching human eye perception:

*   **L (Lightness):** `0%` (pure black) to `100%` (pure white).
*   **C (Chroma):** `0` (grayscale) to `~0.4` (maximum saturation).
*   **H (Hue):** `0` to `360` degrees.

```css
:root {
  /* Predictable, perceptually balanced colors */
  --brand-blue: oklch(0.6 0.18 250);
  --brand-green: oklch(0.6 0.18 145); /* Same perceived lightness as the blue! */
  --brand-yellow: oklch(0.6 0.18 90);  /* Identical perceived contrast against white */
}
```

---

## Relative Color Syntax (CSS Color Level 5)

Generate variants (hover states, borders, semi-transparent overlays) from a single base color token without declaring dozens of static variables:

```css
:root {
  --brand: oklch(0.65 0.2 240);
}

.button {
  background-color: var(--brand);
}

.button:hover {
  /* Increase lightness by 10% */
  background-color: oklch(from var(--brand) calc(l + 0.1) c h);
}

.button:active {
  /* Decrease lightness by 10% */
  background-color: oklch(from var(--brand) calc(l - 0.1) c h);
}

.badge {
  /* Subtle tinted background with 15% opacity */
  background-color: oklch(from var(--brand) l c h / 0.15);
  /* Muted chroma for text */
  color: oklch(from var(--brand) calc(l - 0.2) calc(c * 0.7) h);
}
```

---

## `color-mix()` for Runtime Blending

Blend two colors in a specified interpolation space:

```css
/* Mix 80% surface color with 20% brand color for elevated cards */
.card-elevated {
  background-color: color-mix(in oklch, var(--surface) 80%, var(--brand));
}

/* Semi-transparent ghost button hover */
.ghost-btn:hover {
  background-color: color-mix(in oklch, currentColor 12%, transparent);
}
```

---

## Complete Light/Dark Theming Architecture

The modern way to handle theming eliminates media query duplication and JavaScript theme-class synchronization:

```css
:root {
  /* Declares the document supports both themes */
  color-scheme: light dark;

  /* Theme tokens: light-dark(LIGHT_VAL, DARK_VAL) */
  --bg-canvas: light-dark(oklch(0.98 0.01 240), oklch(0.14 0.02 240));
  --bg-surface: light-dark(oklch(1 0 0), oklch(0.20 0.02 240));
  --text-main: light-dark(oklch(0.18 0.02 240), oklch(0.94 0.01 240));
  --text-muted: light-dark(oklch(0.48 0.03 240), oklch(0.68 0.03 240));
  --border-subtle: light-dark(oklch(0.88 0.01 240), oklch(0.28 0.02 240));
}

body {
  background-color: var(--bg-canvas);
  color: var(--text-main);
}

/* Island theming: easily force an inverted theme on a component */
.dark-panel {
  color-scheme: dark; /* Children automatically evaluate to the dark token */
  background-color: var(--bg-surface);
}
```

---

## High Contrast & Forced Colors Mode

Users with low vision often enable Windows High Contrast Mode or forced colors. Ensure your interactive controls remain visible:

```css
@media (forced-colors: active) {
  /* Invisible custom borders become visible using system keywords */
  .button, .badge, .card {
    border: 1px solid ButtonText;
    outline-color: Highlight;
  }
  
  /* System color tokens: Canvas, CanvasText, ButtonFace, ButtonText, Highlight, HighlightText */
  .accent-indicator {
    background-color: Highlight;
  }
}
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| `hsl(210, 100%, 50%)` | `oklch(0.6 0.22 250)` | HSL lightness is non-uniform; switching hues at identical lightness values causes drastic contrast changes. |
| Duplicate `@media (prefers-color-scheme: dark)` blocks for every selector | `light-dark()` token definitions on `:root` | Media queries scatter theme definitions across files; `light-dark()` groups pairs into single tokens. |
| Hardcoding `#ffffff15` hex alpha | `oklch(from var(--c) l c h / 0.15)` | Hardcoded alpha hexes break when the underlying brand color token is updated. |
| Removing borders on cards/inputs | Use subtle borders (`1px solid var(--border)`) | Borderless UI becomes completely invisible in High Contrast and Forced Colors modes. |
| `rgba(0, 0, 0, 0.5)` overlays | `color-mix(in oklch, black 50%, transparent)` or `oklch(0 0 0 / 0.5)` | Keeps all color declarations aligned in the modern gamut-aware color pipeline. |
