# Neumorphism (Soft UI) Design Principles & Component Skills

A comprehensive, multimodal-ready knowledge base for **Modern Neumorphism (Evolved Soft UI)** engineered specifically for AI coding agents. This package combines strict optical physics rules, OKLCH perceptual color systems, WCAG 2.2 accessibility safeguards, production CSS recipes, and high-resolution visual component sheets in [`./images/`](./images/).

---

## Mission & Purpose

Early 2020-era Neumorphism was notorious for poor usability: low-contrast gray-on-gray text, invisible form inputs, arbitrary shadow directions, broken dark modes, and missing keyboard focus states.

Modern **Evolved Neumorphism** solves every one of those failure modes while preserving the unmistakable tactile beauty of extruded and carved surfaces. This domain equips AI agents to build accessible, production-grade Soft UI interfaces by enforcing:
- **Single Light-Source Physics ($135^\circ$ Top-Left):** Every raised extrusion and carved inset across the viewport obeys one unified directional light vector.
- **Perceptual OKLCH Color Engineering:** Derives top-left highlights and bottom-right shadows dynamically from low-chroma tinted neutrals via CSS Relative Color Syntax (`oklch(from var(--neu-surface) ...)`).
- **Semantic Polarity Hierarchy:** Extrudes elements meant to be *pressed* (buttons, slider thumbs, active tabs) and recesses elements meant to be *filled or tracked* (text inputs, slider grooves, tab troughs).
- **WCAG 2.2 Accessibility Built-In:** Enforces `1px` specular/inset perimeter borders, high-contrast typography ($\ge 7:1$ primary, $\ge 4.5:1$ secondary), offset focus rings (`outline-offset: 4px`), and `@media (prefers-contrast: more)` / `forced-colors: active` fallbacks.

---

## Core Laws of Modern Neumorphism

1. **Never Use Pure White (`#FFF`) or Pure Black (`#000`) Surfaces**  
   Neumorphism requires luminance headroom on *both* sides of the base surface color (`L: 88%–93%` in Light Mode; `L: 20%–25%` in Dark Mode) so top-left highlights are visibly brighter and bottom-right shadows are visibly darker than the canvas.

2. **The $1 : 2$ Offset-to-Blur Ratio**  
   For any shadow offset distance $d$, set the blur radius to approximately $2d$ (e.g., `3px 3px 6px`, `6px 6px 12px`, `12px 12px 24px`).

3. **Respect Shadow Bleed Clearance (`gap >= 2 * blur`)**  
   A `12px 12px 24px` shadow bleeds `36px` outside the element's border box. Maintain generous `gap` (`1.5rem`–`2.5rem`) and container `padding` so shadows never collide or clip against `overflow: hidden`.

4. **Alternate Depth Polarity (No "Wedding Cakes")**  
   Never stack more than two raised layers consecutively. Alternate **Raised Card $\rightarrow$ Inset Well $\rightarrow$ Raised Control** to maintain crisp visual separation.

5. **Never Rotate an Element Carrying a Neumorphic `box-shadow`**  
   Spinning a shadowed element makes the light source orbit in circles. Keep the Neumorphic track stationary and rotate an inner pseudo-element or indicator.

---

## Skill Guides & Multimodal Visual Index

| Guide | Visual Reference Image | Key Topics & Components |
| :--- | :--- | :--- |
| **[`neumorphism_foundations_and_tokens_skill_guide.md`](./neumorphism_foundations_and_tokens_skill_guide.md)** | [`images/neumorphism_surface_states.jpg`](./images/neumorphism_surface_states.jpg) | $135^\circ$ single light-source physics, 4 fundamental surface states (Extruded Flat, Inset Recessed, Convex Dome, Concave Scoop), `1:2` blur math, elevation & radius tokens. |
| **[`neumorphism_color_palettes_and_theming_skill_guide.md`](./neumorphism_color_palettes_and_theming_skill_guide.md)** | [`images/neumorphism_color_palettes.jpg`](./images/neumorphism_color_palettes.jpg) | OKLCH lightness headroom math, 4 curated theme palettes (Cool Porcelain, Warm Alabaster Sand, Sage Mist, Obsidian Slate Dark), `light-dark()` + Relative Color Syntax, colored accent shadows. |
| **[`neumorphism_layout_and_cards_skill_guide.md`](./neumorphism_layout_and_cards_skill_guide.md)** | [`images/neumorphism_cards_and_layout.jpg`](./images/neumorphism_cards_and_layout.jpg) | Shadow bleed clearance formula, Bento/dashboard grids, alternating depth polarity (Raised $\rightarrow$ Inset $\rightarrow$ Raised), `2px` etched seam dividers, 30% Soft-UI density cap. |
| **[`neumorphism_button_skill_guide.md`](./neumorphism_button_skill_guide.md)** | [`images/neumorphism_buttons.jpg`](./images/neumorphism_buttons.jpg) | 4-state button physics (Idle $\rightarrow$ Hover $\rightarrow$ Active Inset $\rightarrow$ Focus), circular icon buttons, high-contrast Primary Accent CTA buttons, segmented button groups. |
| **[`neumorphism_forms_and_inputs_skill_guide.md`](./neumorphism_forms_and_inputs_skill_guide.md)** | [`images/neumorphism_forms_and_inputs.jpg`](./images/neumorphism_forms_and_inputs.jpg) | Recessed concave text inputs & auto-sizing textareas, composite search bars (`:focus-within`), `:user-valid` / `:user-invalid` glow rings, native `appearance: none` checkboxes & radios. |
| **[`neumorphism_sliders_and_toggles_skill_guide.md`](./neumorphism_sliders_and_toggles_skill_guide.md)** | [`images/neumorphism_sliders_and_toggles.jpg`](./images/neumorphism_sliders_and_toggles.jpg) | Tactile `<input type="range">` with recessed groove + filled progress channel + raised spherical thumb, binary `role="switch"` toggles, native vertical faders, rotary knobs. |
| **[`neumorphism_loader_and_progress_skill_guide.md`](./neumorphism_loader_and_progress_skill_guide.md)** | [`images/neumorphism_loaders_and_progress.jpg`](./images/neumorphism_loaders_and_progress.jpg) | Stationary carved donut spinners with rotating conic arcs, circular percentage gauges, carved linear progress bars, concentric tactile ripples, `prefers-reduced-motion` safeguards. |
| **[`neumorphism_navigation_and_accessibility_skill_guide.md`](./neumorphism_navigation_and_accessibility_skill_guide.md)** | [`images/neumorphism_navigation.jpg`](./images/neumorphism_navigation.jpg) | Recessed segmented pill tab bars (`role="tablist"`), floating navbars, pagination strips, breadcrumb troughs, and the 5 mandatory WCAG 2.2 accessibility safeguards. |

---

## How AI Agents Must Use This Package

1. **Read Foundations & Palettes First:** Before generating any Soft UI stylesheet, load [`neumorphism_foundations_and_tokens_skill_guide.md`](./neumorphism_foundations_and_tokens_skill_guide.md) and [`neumorphism_color_palettes_and_theming_skill_guide.md`](./neumorphism_color_palettes_and_theming_skill_guide.md) to establish the root `:root` token system.
2. **Inspect Component Reference Images:** If you are a multimodal agent capable of viewing images, inspect the corresponding `.jpg` file in [`./images/`](./images/) alongside the component skill guide to verify visual proportions, contrast, and shadow softness.
3. **Audit Every Component Against the "Do Not" Table:** Before outputting HTML/CSS, verify that no anti-patterns (such as `#ffffff` canvases, extruded text inputs, rotating `box-shadow` spinners, or missing `:focus-visible` outlines) are present.
