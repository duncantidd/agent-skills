# Japandi (Scandi-Minimalism) — AI Agent Skill Package

A comprehensive, multimodal design system and CSS engineering package for building **Japandi (Scandinavian Minimalism × Japanese *Wabi-Sabi* & *Ma*) Web UIs**. Engineered specifically for AI coding agents, each skill guide pairs **zero-dependency modern CSS (`oklch()`, Relative Color Syntax `oklch(from ...)`, `light-dark()`, `subgrid`, `@starting-style`, flow-relative logical properties, and WCAG 2.2 accessibility safeguards)** with **high-resolution visual component specification sheets** in [`./images/`](./images/).

---

## Quick Routing Matrix for AI Agents

Load only the specific skill guide(s) and visual reference sheet(s) needed for your current UI task:

| Task / Component | Skill Guide | Visual Reference Sheet |
| :--- | :--- | :--- |
| **Core Tokens, *Ma* Space, *Wabi-Sabi* Materiality, *Shibui* Elevation & Ceramic Radius Scale** | [`japandi_foundations_and_tokens_skill_guide.md`](./japandi_foundations_and_tokens_skill_guide.md) | [`./images/japandi_foundations.jpg`](./images/japandi_foundations.jpg) |
| **Color Palettes (Kyoto & Copenhagen, Nordic Birch, Nara Clay, Shou Sugi Ban Dark Mode) & 70/20/10 Rule** | [`japandi_color_palettes_and_theming_skill_guide.md`](./japandi_color_palettes_and_theming_skill_guide.md) | [`./images/japandi_color_palettes.jpg`](./images/japandi_color_palettes.jpg) |
| **Editorial Serif (`300–400`) + Humanist Sans Pairing, Vertical *Tategaki* Markers, Pull Quotes & Ceramic Tags** | [`japandi_typography_and_editorial_hierarchy_skill_guide.md`](./japandi_typography_and_editorial_hierarchy_skill_guide.md) | [`./images/japandi_typography_and_hierarchy.jpg`](./images/japandi_typography_and_hierarchy.jpg) |
| **Asymmetric 7/5 Heroes, Active *Ma* Spacing, `1px` *Shoji* Partition Grids & Matted Gallery `subgrid` Cards** | [`japandi_layout_ma_spacing_and_shoji_grids_skill_guide.md`](./japandi_layout_ma_spacing_and_shoji_grids_skill_guide.md) | [`./images/japandi_layout_and_cards.jpg`](./images/japandi_layout_and_cards.jpg) |
| **Grounded Sumi Ink & Matcha Buttons, Stone Hairline Buttons, Quiet Links, Hygge Motion & Ceramic Filter Pills** | [`japandi_button_and_action_skill_guide.md`](./japandi_button_and_action_skill_guide.md) | [`./images/japandi_buttons.jpg`](./images/japandi_buttons.jpg) |
| **Recessed Washi-Paper & Bottom-Rule Inputs, Matcha Focus, Kiln Terracotta `:user-invalid` & Ceramic Checkboxes** | [`japandi_forms_and_inputs_skill_guide.md`](./japandi_forms_and_inputs_skill_guide.md) | [`./images/japandi_forms_and_inputs.jpg`](./images/japandi_forms_and_inputs.jpg) |
| **Pebble & Bamboo `role="switch"` Toggles, `2px` Horizon Range Sliders, *Enso* Spinners & Ritual Steppers** | [`japandi_toggles_sliders_and_zen_loaders_skill_guide.md`](./japandi_toggles_sliders_and_zen_loaders_skill_guide.md) | [`./images/japandi_toggles_and_loaders.jpg`](./images/japandi_toggles_and_loaders.jpg) |
| **Translucent *Shoji* Headers, Native `<dialog>` Sanctuary Modals, Slide-Over Drawers, Toasts & WCAG 2.2 Laws** | [`japandi_navigation_drawers_and_accessibility_skill_guide.md`](./japandi_navigation_drawers_and_accessibility_skill_guide.md) | [`./images/japandi_navigation_and_dialogs.jpg`](./images/japandi_navigation_and_dialogs.jpg) |

---

## The 5 Non-Negotiable Laws of Japandi UI

1. **The Law of *Ma* (間 — Active Negative Space):** Empty space is the primary structural material. At least `40%` of the viewport must remain open warm linen canvas (`--japandi-canvas`), enforced via fluid `clamp(3rem, 6vw, 5.5rem)` section intervals and asymmetric grid gaps.
2. **The 70 / 20 / 10 Warm Mineral Harmony Rule:** Never use sterile pure white (`#FFFFFF`) or pitch black (`#000000`). Anchor **70%** in Warm Washi Linen (`#F5F2EB` / `#181614` Charred Hinoki in dark mode), **20%** in Honed Travertine & Smoked Bamboo (`#FAF8F3` / `#EAE4D8`), and **10%** in Grounded Sumi Charcoal (`#23211E`) and Botanical Pigments (Moss Matcha `#4A5D4E`, Kiln Terracotta `#B86B4B`, Boro Indigo `#3B4D5E`).
3. **Whisper-Quiet *Shibui* Elevation & `1px` *Shoji* Joinery:** Reject heavy Material drop shadows and thick Neo-Brutalist borders. Define cards and grid partitions using `1px solid` warm stone hairlines (`#DDD6C7`) paired with ultra-diffused, low-opacity ambient paper shadows (`0 12px 32px -8px oklch(0.24 0.02 72 / 0.06)`).
4. **Dual-Voice Poetic Typography (Light Editorial Serif + Humanist Sans):** Set display and section headings in a light/regular high-contrast Editorial Serif (`Cormorant Garamond`, `Instrument Serif` at weight `300–400`, never `800+`) paired with breathable Humanist Sans body copy (`Plus Jakarta Sans`, `Inter` at `line-height: 1.75`, `max-inline-size: 60ch`).
5. **Hygge Slow-Deceleration Motion & WCAG 2.2 Contrast:** Transitions glide with unhurried, grounded deceleration (`cubic-bezier(0.22, 1, 0.36, 1)` over `320ms–450ms`, `-1px` hover lift—never bouncy springs), while maintaining `> 12:1` AAA Sumi Ink text contrast and `>= 3:1` interactive form boundaries.
