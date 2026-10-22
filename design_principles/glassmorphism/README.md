# Glassmorphism (Spatial & Frosted Glass) — AI Agent Skill Package

A comprehensive, multimodal design system and CSS engineering package for building **Modern Glassmorphic & Spatial Computing (visionOS / Refractive Frosted Glass) Web UIs**. Engineered specifically for AI coding agents, each skill guide pairs **zero-dependency modern CSS (`backdrop-filter: blur() saturate()`, $135^\circ$ `border-box` specular rim gradients, `oklch()`, Relative Color Syntax `oklch(from ...)`, `@starting-style`, and WCAG 2.2 `prefers-reduced-transparency` safeguards)** with **high-resolution visual component specification sheets** in [`./images/`](./images/).

---

## Quick Routing Matrix for AI Agents

Load only the specific skill guide(s) and visual reference sheet(s) needed for your current UI task:

| Task / Component | Skill Guide | Visual Reference Sheet |
| :--- | :--- | :--- |
| **Core Tokens, 4 Optical Laws, $135^\circ$ Specular Rim Borders, 3-Tier Z-Depth & Concentric Radii** | [`glassmorphism_foundations_and_tokens_skill_guide.md`](./glassmorphism_foundations_and_tokens_skill_guide.md) | [`./images/glassmorphism_foundations.jpg`](./images/glassmorphism_foundations.jpg) |
| **OKLCH Mesh Substrates & Palettes (Cosmic Aurora, Cupertino Daylight, Emerald Bioluminescence, Sunset Prism)** | [`glassmorphism_color_palettes_and_theming_skill_guide.md`](./glassmorphism_color_palettes_and_theming_skill_guide.md) | [`./images/glassmorphism_color_palettes.jpg`](./images/glassmorphism_color_palettes.jpg) |
| **Optical Legibility (`font-weight >= 450`), Specular Top-Rim KPI Pods & Prismatic Status Badges** | [`glassmorphism_typography_badges_and_optical_legibility_skill_guide.md`](./glassmorphism_typography_badges_and_optical_legibility_skill_guide.md) | [`./images/glassmorphism_typography_and_badges.jpg`](./images/glassmorphism_typography_and_badges.jpg) |
| **3-Layer Spatial Windows (No Blur-on-Blur), Concentric Radius Math, Bento Grids & Media Lens Cards** | [`glassmorphism_layout_cards_and_spatial_hierarchy_skill_guide.md`](./glassmorphism_layout_cards_and_spatial_hierarchy_skill_guide.md) | [`./images/glassmorphism_layout_and_cards.jpg`](./images/glassmorphism_layout_and_cards.jpg) |
| **Specular Capsule Buttons, Iridescent Primary CTAs, 4-State Gaze/Hover Physics & Segmented Pill Bars** | [`glassmorphism_button_and_action_skill_guide.md`](./glassmorphism_button_and_action_skill_guide.md) | [`./images/glassmorphism_buttons.jpg`](./images/glassmorphism_buttons.jpg) |
| **Recessed Smoked-Glass Input Wells, Luminous Focus Halos, `⌘K` Spotlight Bars, Checkboxes & Radio Orbs** | [`glassmorphism_forms_and_inputs_skill_guide.md`](./glassmorphism_forms_and_inputs_skill_guide.md) | [`./images/glassmorphism_forms_and_inputs.jpg`](./images/glassmorphism_forms_and_inputs.jpg) |
| **Liquid Glass `role="switch"` Toggles, Frosted Range Sliders, Conic Spinners, Shimmers & Progress Tubes** | [`glassmorphism_toggles_sliders_and_luminous_loaders_skill_guide.md`](./glassmorphism_toggles_sliders_and_luminous_loaders_skill_guide.md) | [`./images/glassmorphism_toggles_and_loaders.jpg`](./images/glassmorphism_toggles_and_loaders.jpg) |
| **Detached Spatial Capsule Docks, Native `<dialog>` Spatial Modals, Frosted Toasts & 5 WCAG 2.2 / GPU Laws** | [`glassmorphism_navigation_dialogs_and_accessibility_skill_guide.md`](./glassmorphism_navigation_dialogs_and_accessibility_skill_guide.md) | [`./images/glassmorphism_navigation_and_dialogs.jpg`](./images/glassmorphism_navigation_and_dialogs.jpg) |

---

## The 5 Non-Negotiable Laws of Glassmorphism

1. **Refractive Blur + Chromatic Saturation (`backdrop-filter: blur(16px–28px) saturate(160%–180%)`):** Never use `blur()` alone—always pair it with `saturate(160%+)` over an atmospheric multi-radial aurora mesh or rich media substrate so underlying colors refract vibrantly through the pane.
2. **$135^\circ$ Directional Specular Rim Borders (`background-clip: padding-box, border-box`):** Real glass catches light along its upper-left cut edge (`oklch(1 0 0 / 0.48)`) and fades toward the bottom-right (`oklch(1 0 0 / 0.04)`), reinforced by a `1px` top inner glint (`inset 0 1px 0 oklch(1 0 0 / 0.35)`).
3. **The 3-Tier Spatial Z-Depth & GPU Cap (Max 2 Stacked Blurs):** Never nest 3+ `backdrop-filter` elements. Use **Tier 1** (`blur(20px)`) for outer windows/cards, **Tier 2** (`oklch(0 0 0 / 0.45)` recessed smoked-glass wells with **zero nested blur**) for inner sidebars/inputs, and **Tier 3** (`blur(28px)`) for floating docks and modals.
4. **Minimum Opacity Floor & Optical Typography (`>= 4.5:1` Contrast):** Anchor dark-mode glass with a Smoked Quartz tint floor (`alpha >= 0.38`) and light-mode glass with Frosted Silica (`alpha >= 0.58`), paired with `font-weight >= 450` and subtle optical `text-shadow` so bright background orbs never wash out foreground text.
5. **Mandatory `@media (prefers-reduced-transparency: reduce)` & `forced-colors` Fallbacks:** Automatically replace translucent glass with `>= 95%` opaque solid surfaces and disable `backdrop-filter` when requested by user OS accessibility preferences.
