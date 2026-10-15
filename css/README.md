# CSS Native Skills & Best Practices

A zero-dependency, platform-native CSS knowledge base engineered specifically for AI coding agents. This directory provides production-ready decision matrices, modern code patterns, and explicit anti-pattern rules across the entire modern CSS specification.

---

## Mission & Purpose

Modern CSS has evolved rapidly. Many LLMs and AI coding assistants still default to obsolete hacks from the 2010s:
- Using `top: 50%; left: 50%; transform: translate(-50%, -50%)` instead of `place-content: center`
- Injecting JavaScript for basic parent styling instead of `:has()`
- Heavy Sass nesting and bloated resets instead of Cascade Layers (`@layer`) and `:where()`
- Inflexible `100vh` units instead of dynamic viewport units (`100dvh`)
- Hardcoded physical directions (`margin-left`) instead of logical properties (`margin-inline-start`)

The purpose of this directory is to equip AI agents with **authoritative, modern, baseline-supported CSS capabilities** so they write clean, responsive, accessible, and high-performance styles without unnecessary libraries or legacy debt.

---

## Core Engineering Principles

1. **Platform Native First**  
   Leverage modern browser capabilities (`@starting-style`, Popover API, `<dialog>`, Subgrid, `:has()`, `interpolate-size`) before reaching for JavaScript or external packages.

2. **Internationalization by Default**  
   Use flow-relative logical properties (`inline-size`, `block-size`, `margin-inline`, `padding-block`, `inset`) so that layouts instantly support Right-to-Left (RTL) languages and vertical writing modes.

3. **Perceptually Uniform Color & Dynamic Theming**  
   Use OKLCH for predictable lightness and chroma across all hues. Leverage `light-dark()` and Relative Color Syntax (`oklch(from var(--c) ... )`) to eliminate duplicate dark-mode rules and complex theme scripts.

4. **Container & Intrinsic Responsiveness**  
   Decouple components from the viewport using Container Queries (`@container`), fluid typography (`clamp()`), and intrinsic form sizing (`field-sizing: content`).

5. **Low-Specificity Cascade Management**  
   Control the cascade using Cascade Layers (`@layer`), zero-specificity defaults (`:where()`), and native scoping (`@scope`) rather than specificity wars and `!important`.

6. **Accessibility & Preference Respect Built-In**  
   Enforce visible keyboard focus (`:focus-visible`), resilient screen-reader utilities (`.sr-only`), tap-target ergonomics (`pointer: coarse`), and respect for user preferences (`prefers-reduced-motion`, `forced-colors`).

---

## Directory Index

| Guide | Description | Key Modern Features |
| :--- | :--- | :--- |
| **[`css_layout_skill_guide.md`](./css_layout_skill_guide.md)** | Modern 1D/2D layouts, alignment, and viewport adaptation | Flexbox min-size trap (`min-inline-size: 0`), Grid RAM pattern, Subgrid, Container Queries, Logical Properties, Viewport units (`dvh`/`svh`), Centering. |
| **[`css_architecture_skill_guide.md`](./css_architecture_skill_guide.md)** | Scalable stylesheet architecture and specificity control | Cascade Layers (`@layer`), Specificity zeroing (`:where()`), Native Scoping (`@scope`), Native Nesting, Minimal Modern Reset, Typed Tokens (`@property`). |
| **[`css_typography_skill_guide.md`](./css_typography_skill_guide.md)** | Fluid type scaling, typesetting, and readability ergonomics | Fluid typography (`clamp()`), `text-wrap: balance` & `pretty`, multi-line `line-clamp`, `overflow-wrap: break-word`, `ch` reading measure, Variable Fonts, `font-display: swap`. |
| **[`css_animation_skill_guide.md`](./css_animation_skill_guide.md)** | High-performance visual motion and state transitions | Scroll-driven animations (`animation-timeline`), View Transitions API, `@starting-style` + `allow-discrete`, CSS Anchor Positioning, `linear()` spring easing, composite-only GPU animations. |
| **[`css_colors_skill_guide.md`](./css_colors_skill_guide.md)** | Perceptually uniform color spaces, blending, and theming | OKLCH color space, Relative Color Syntax (`oklch(from var(--c) ...)`), `color-mix()`, Native `light-dark()`, High Contrast & Windows `forced-colors`. |
| **[`css_selectors_and_state_skill_guide.md`](./css_selectors_and_state_skill_guide.md)** | Relational selectors and modern state management | Relational selector (`:has()`), User-aware form states (`:user-valid`/`:user-invalid`), Keyboard focus (`:focus-visible`), `:focus-within`, floating labels via `:placeholder-shown`. |
| **[`css_accessibility_skill_guide.md`](./css_accessibility_skill_guide.md)** | Inclusive design, screen reader utilities, and user preferences | Zero-side-effect `.sr-only` pattern, High-contrast focus rings, `prefers-reduced-motion`, `pointer: coarse` tap target expansion (44x44px), 200% text zoom resilience. |
| **[`css_components_and_forms_skill_guide.md`](./css_components_and_forms_skill_guide.md)** | Standard HTML5 UI components and modern form styling | Native `<dialog>` + `::backdrop`, Popover API (`popover`), Exclusive accordions (`<details name>`), Intrinsic form sizing (`field-sizing: content`), `accent-color`. |

---

## Agent Usage Instructions

When generating or refactoring CSS:
1. **Identify the Task Domain**: Match the styling task to the relevant guide above.
2. **Consult the Decision Matrix**: Check the guide's Decision Matrix table to choose the right modern CSS tool for the specific goal.
3. **Verify Anti-Patterns**: Check the guide's **Do Not** table to ensure obsolete patterns (e.g. `!important`, physical coordinates, `100vh`, legacy reset bloat) are not introduced.
4. **Prefer Compositor-Driven & GPU-Accelerated Properties**: When animating, restrict properties to `transform`, `opacity`, and `clip-path`.
