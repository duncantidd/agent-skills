# Swiss Grid (International Typographic Style) Design Principles & Component Skills

A comprehensive, multimodal-ready knowledge base for **Swiss Grid (International Typographic Style)** web UIs engineered specifically for AI coding agents. Designed for text-heavy editorial publications, magazines, architectural portfolios, documentation, and research archives, this package combines mathematical CSS Grid/Subgrid rules, Neo-Grotesque typographic systems, OKLCH paper-and-ink palettes, and high-resolution visual specification sheets in [`./images/`](./images/).

---

## Mission & Purpose

Pioneered in Switzerland in the 1950s by Josef Müller-Brockmann, Karl Gerstner, and Emil Ruder, the **International Typographic Style** treats the structural grid and objective typography as the sole ornament of the page.

When AI agents attempt to build editorial or magazine layouts without strict guidance, they often default to generic SaaS clichés—centered hero text over darkened stock photos, rounded `12px` drop-shadow cards, timid typographic scales, and multi-colored pill badges. This package equips AI agents to build authoritative Swiss Grid interfaces by enforcing:
- **Exposed Mathematical Grid Architecture:** Visible `1px` cell hairlines (via the seamless `gap: 1px` background-bleed technique) paired with heavy `2px–4px` horizontal section rules.
- **Extreme Typographic Scale Contrast:** Monumental, tight-leaded Neo-Grotesque display headers (`line-height: 0.92`) juxtaposed against `62ch` running prose and `12px` uppercase monospace coordinate markers (`[COL 01 // ROW 02]`).
- **Flush-Left, Ragged-Right Rigor:** Strict `text-align: start` alignment locked to vertical grid tracks—never centered body copy or river-prone `text-align: justify`.
- **90 / 10 Monochromatic + Single Signal Accent Palette:** Warm archival paper (`#F5F4F0`) and carbon ink (`#111111`) punctuated by a single surgical signal hue (e.g., Swiss International Red `#E4002B` or Klein Cobalt `#0038A8`).
- **Zero-Radius Orthogonality & Hard-Edge Inversion:** Strict $90^\circ$ corners (`border-radius: 0`) across all elements, with interactive states communicated via instant full-cell polarity inversion rather than drop shadows.

---

## Core Laws of Swiss Grid Web Design

1. **Zero Radius Everywhere (`border-radius: 0`)**  
   Every grid cell, card, image frame, button, input, checkbox, and focus ring uses strict $90^\circ$ corners so vertical and horizontal rules intersect cleanly.

2. **Use the Background-Bleed Technique for Exposed `1px` Hairlines**  
   Never put `border: 1px solid` on adjacent sibling grid items (which doubles internal seams to `2px`). Set `background-color: var(--swiss-ink); gap: 1px; border: 1px solid var(--swiss-ink)` on the parent grid and `background-color: var(--swiss-paper)` on child cells.

3. **Reserve 25%–35% of Grid Modules as Asymmetric Whitespace**  
   Empty grid cells are active structural counterweights. Pair an 8-column headline or image plate with a deliberate 4-column open or metadata cell.

4. **Align Multi-Column Feeds with CSS `subgrid`**  
   Use `grid-template-rows: subgrid` on multi-column article cards so category indices, headlines, excerpts, and bottom action bars lock horizontally across adjacent columns.

5. **Enforce `font-variant-numeric: tabular-nums` on All Figures**  
   Index numbers (`01`, `02`), issue dates, timestamps, reading percentages, and data tables must use tabular numerals so vertical columns never jitter.

---

## Skill Guides & Multimodal Visual Index

| Guide | Visual Reference Image | Key Topics & Components |
| :--- | :--- | :--- |
| **[`swiss_grid_foundations_and_tokens_skill_guide.md`](./swiss_grid_foundations_and_tokens_skill_guide.md)** | [`images/swiss_grid_foundations.jpg`](./images/swiss_grid_foundations.jpg) | 12-column mathematical grid tokens, seamless `gap: 1px` exposed hairline technique, `2px–4px` structural rules, zero-radius law, coordinate markers (`[COL 01 // ROW 02]`). |
| **[`swiss_grid_typography_and_hierarchy_skill_guide.md`](./swiss_grid_typography_and_hierarchy_skill_guide.md)** | [`images/swiss_grid_typography.jpg`](./images/swiss_grid_typography.jpg) | Neo-Grotesque + Technical Monospace pairing, extreme `clamp()` scale contrast, tight `0.92` display leading, flush-left ragged-right rule, scholarly marginalia/sidenote rails, `4px` top-ruled pull quotes. |
| **[`swiss_grid_color_palettes_and_theming_skill_guide.md`](./swiss_grid_color_palettes_and_theming_skill_guide.md)** | [`images/swiss_grid_color_palettes.jpg`](./images/swiss_grid_color_palettes.jpg) | 90/10 monochrome + Single Signal Accent rule, 4 canonical palettes (*Zurich Archival Standard*, *Basel Editorial Newsprint*, *Geneva Industrial Concrete*, *Midnight Monolith Dark*), hard-edge hover inversions. |
| **[`swiss_grid_editorial_layout_and_cards_skill_guide.md`](./swiss_grid_editorial_layout_and_cards_skill_guide.md)** | [`images/swiss_grid_editorial_layout.jpg`](./images/swiss_grid_editorial_layout.jpg) | Asymmetric magazine hero spreads (8-col display + 4-col brief + framed monochrome plate), 3-column `subgrid` article feeds, full-cell hover polarity inversion. |
| **[`swiss_grid_button_and_action_skill_guide.md`](./swiss_grid_button_and_action_skill_guide.md)** | [`images/swiss_grid_buttons.jpg`](./images/swiss_grid_buttons.jpg) | Zero-radius architectural buttons, Primary Signal Red CTAs, Full-Width Grid-Cell Action Bars (`01 | TITLE | →`), bracketed technical micro-links (`[ READ ESSAY → ]`), segmented category filter strips. |
| **[`swiss_grid_forms_and_inputs_skill_guide.md`](./swiss_grid_forms_and_inputs_skill_guide.md)** | [`images/swiss_grid_forms_and_inputs.jpg`](./images/swiss_grid_forms_and_inputs.jpg) | Architectural `2px` bottom-rule inputs, numbered monospace catalog labels (`01.1 // FULL NAME`), seamless cell-framed subscription forms, `:user-invalid` red rule + error code badge, zero-radius square checkboxes & radios. |
| **[`swiss_grid_tables_and_data_index_skill_guide.md`](./swiss_grid_tables_and_data_index_skill_guide.md)** | [`images/swiss_grid_tables_and_index.jpg`](./images/swiss_grid_tables_and_index.jpg) | Interactive full-row hover inversion archive indexes, exposed `1px` hairline specification tables with `tabular-nums`, monolithic KPI callout cells. |
| **[`swiss_grid_navigation_and_status_skill_guide.md`](./swiss_grid_navigation_and_status_skill_guide.md)** | [`images/swiss_grid_navigation_and_status.jpg`](./images/swiss_grid_navigation_and_status.jpg) | Cell-partitioned editorial mastheads, sticky numbered Table of Contents (TOC) rail with `8px` Signal Red active bar, `scroll()` reading progress rules, step tickers, and WCAG 2.2 zero-radius focus rings. |

---

## How AI Agents Must Use This Package

1. **Read Foundations, Typography & Palettes First:** Before generating any editorial or magazine layout, load [`swiss_grid_foundations_and_tokens_skill_guide.md`](./swiss_grid_foundations_and_tokens_skill_guide.md), [`swiss_grid_typography_and_hierarchy_skill_guide.md`](./swiss_grid_typography_and_hierarchy_skill_guide.md), and [`swiss_grid_color_palettes_and_theming_skill_guide.md`](./swiss_grid_color_palettes_and_theming_skill_guide.md).
2. **Inspect Component Reference Sheets:** If you are a multimodal agent capable of viewing images, inspect the corresponding `.jpg` specification sheet in [`./images/`](./images/) to verify column proportions, hairline junctions, and typographic contrast.
3. **Audit Against the "Do Not" Table:** Before outputting HTML/CSS, verify that no anti-patterns (such as rounded corners, drop shadows, centered/justified body copy, or doubled `2px` cell borders) are present.
