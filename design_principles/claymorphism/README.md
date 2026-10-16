# Claymorphism — AI Agent Skill Package

A comprehensive, multimodal design system and CSS engineering package for building **Modern Claymorphic (3D Matte Clay & Marshmallow) Web UIs**. Engineered specifically for AI coding agents, each skill guide pairs **zero-dependency modern CSS (`oklch()`, Relative Color Syntax `oklch(from ...)`, `light-dark()`, `@starting-style`, flow-relative logical properties, and WCAG 2.2 accessibility safeguards)** with **high-resolution visual component specification sheets** in [`./images/`](./images/).

---

## Quick Routing Matrix for AI Agents

Load only the specific skill guide(s) and visual reference sheet(s) needed for your current UI task:

| Task / Component | Skill Guide | Visual Reference Sheet |
| :--- | :--- | :--- |
| **Core Tokens, 3-Layer Shadow Equation, Pressed Dough Wells & Inflation Radius Scale** | [`claymorphism_foundations_and_tokens_skill_guide.md`](./claymorphism_foundations_and_tokens_skill_guide.md) | [`./images/claymorphism_foundations.jpg`](./images/claymorphism_foundations.jpg) |
| **Color Palettes (Cotton Candy Studio, Matcha & Terracotta, Toybox Pop, Midnight Velvet Dark)** | [`claymorphism_color_palettes_and_theming_skill_guide.md`](./claymorphism_color_palettes_and_theming_skill_guide.md) | [`./images/claymorphism_color_palettes.jpg`](./images/claymorphism_color_palettes.jpg) |
| **Rounded Display Typography, KPI Metric Pods, Puffy 3D Pill Badges & Clay Orbs** | [`claymorphism_typography_badges_and_3d_accents_skill_guide.md`](./claymorphism_typography_badges_and_3d_accents_skill_guide.md) | [`./images/claymorphism_typography_and_badges.jpg`](./images/claymorphism_typography_and_badges.jpg) |
| **Inflated Product & Pricing Cards, Nested Depth Polarity & Bento Grid Shadow Clearance** | [`claymorphism_layout_cards_and_floating_surfaces_skill_guide.md`](./claymorphism_layout_cards_and_floating_surfaces_skill_guide.md) | [`./images/claymorphism_cards_and_layout.jpg`](./images/claymorphism_cards_and_layout.jpg) |
| **Squishy Marshmallow Buttons, 4-State Physics, 3D Icon Cushions & Segmented Pill Bars** | [`claymorphism_button_skill_guide.md`](./claymorphism_button_skill_guide.md) | [`./images/claymorphism_buttons.jpg`](./images/claymorphism_buttons.jpg) |
| **Pressed Dough-Well Inputs, Focus Inflation, Composite Search Wells, Checkboxes & Radios** | [`claymorphism_forms_and_inputs_skill_guide.md`](./claymorphism_forms_and_inputs_skill_guide.md) | [`./images/claymorphism_forms_and_inputs.jpg`](./images/claymorphism_forms_and_inputs.jpg) |
| **Marshmallow Toggle Switches, Inflated Tube Range Sliders, 3D Tube Progress & Bouncing Orbs** | [`claymorphism_toggles_sliders_and_loaders_skill_guide.md`](./claymorphism_toggles_sliders_and_loaders_skill_guide.md) | [`./images/claymorphism_toggles_and_loaders.jpg`](./images/claymorphism_toggles_and_loaders.jpg) |
| **Floating Cloud Pill Navbars, Native `<dialog>` 3D Clay Modals, Toast Bubbles & WCAG 2.2** | [`claymorphism_navigation_dialogs_and_accessibility_skill_guide.md`](./claymorphism_navigation_dialogs_and_accessibility_skill_guide.md) | [`./images/claymorphism_navigation_and_dialogs.jpg`](./images/claymorphism_navigation_and_dialogs.jpg) |

---

## The 5 Non-Negotiable Laws of Claymorphism

1. **The 3-Layer Shadow Stack:** Every inflated clay surface combines **1 soft outer ambient drop shadow** (`0 20px 35px -10px var(--_drop)`) with **2 opposing `inset` rim shadows**: a top-left inner highlight (`inset 8px 8px 16px 0 var(--_hi)`) and a hue-matched bottom-right inner volume shade (`inset -8px -8px 16px 0 var(--_lo)`).
2. **Super-Rounded Inflated Geometry ($R \ge 2 \times \text{inner-offset}$):** Sharp corners (`< 12px`) pinch inner shadows into harsh diagonal creases. Use `16px`–`22px` for chips/inputs, `26px`–`40px` for buttons/cards, and `9999px` for pills.
3. **Chromatic Inner Volume (Never Muddy Gray):** Derive bottom-right inner shades using CSS Relative Color Syntax (`oklch(from var(--_bg) calc(l - 0.16) calc(c * 1.2) h / 0.62)`) so pastel pink, mint, butter, and lilac clay retains vibrant 3D underside saturation.
4. **Tactile Depth Polarity (Inflated Clay vs. Pressed Dough Wells):** Clickable controls (buttons, active pills, checked switches) are **raised 3D clay**; fillable receptacles (text inputs, slider tracks, segmented bar tracks) are **recessed dough wells** (`inset` top-left shade + bottom-right rim).
5. **Deep Plum Ink (`#1E1B3A`) on Pastel Clay (`> 10:1` AAA Contrast):** Never place white text (`#FFFFFF`) on light pastel clay (`L > 0.75`). Always pair pastel surfaces with deep plum-indigo (`#1E1B3A`) or dark hue-matched ink.
