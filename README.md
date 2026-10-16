# Agent Skills

A curated repository of production-ready skill guides, decision matrices, architectural patterns, and anti-pattern rules engineered specifically for AI coding agents to reference and apply.

---

## Purpose

Large language models often default to obsolete patterns, legacy polyfills, and outdated techniques learned from vast historical training corpuses. 

The `agent-skills` repository acts as an external memory and authoritative knowledge base for agents, providing clear rules on:
- What modern, platform-native tools to select for any given goal.
- How to structure clean, maintainable, zero-dependency implementations.
- Which legacy anti-patterns and performance traps to strictly avoid.

---

## Setup & Integration Guide

The recommended way to use `agent-skills` across your projects is the **Router Pattern (Progressive Disclosure)**:
Rather than dumping thousands of lines of rules into your root agent instructions (which wastes tokens and degrades reasoning on unrelated tasks), you drop this repository into a `.skills/` folder and instruct the agent to load guides on-demand.

### Step 1: Add `agent-skills` to Your Project

Choose the installation method that fits your workflow:

#### Option A: Git Submodule (Recommended for Multi-Project Sync)
Keeps skills centralized so a single command updates skills across all your repositories:
```bash
# Add to your project root under .skills/
git submodule add https://github.com/<your-username>/agent-skills.git .skills

# When cloning a project that already uses .skills:
git clone --recurse-submodules <project-url>

# To pull the latest skills updates at any time:
git submodule update --remote --merge
```

#### Option B: Direct Copy / Vendoring (Zero Git Overhead)
Ideal for team repositories or CI/CD pipelines where you want zero submodule dependencies:
```bash
# Using degit to download without git history
npx degit <your-username>/agent-skills .skills

# Or simply clone and copy the directory:
git clone https://github.com/<your-username>/agent-skills.git temp-skills
cp -r temp-skills .skills
rm -rf temp-skills
```

#### Option C: Global Machine-Level Integration
If you prefer not to commit skills into individual project repositories, keep this repo in your user home directory:
```bash
git clone https://github.com/<your-username>/agent-skills.git ~/agent-skills
```
Then reference `~/agent-skills/` from your global `~/.claude/CLAUDE.md`.

---

### Step 2: Configure Your Project's Agent Router

Add the following router directive to your target project's root [**`CLAUDE.md`**](./CLAUDE.md):

```markdown
## Domain Skills & Modern Standards

Before writing, refactoring, or reviewing frontend styling or architecture, you MUST consult the skills library in `.skills/`:

1. **CSS Tasks**: Consult `.skills/css/README.md` to identify the relevant guide (e.g. layout, colors, typography, state).
2. **JavaScript Tasks**: Consult `.skills/javascript/README.md` for standard library utilities, async patterns, DOM/events, and error causation.
3. **Wizz.js Tasks**: Consult `.skills/wizz/README.md` for component architecture, reactivity, template directives, and deployment.
4. **Design Systems (Neumorphism, Swiss Grid & Neo-Brutalism)**: Consult `.skills/design_principles/README.md` (`.skills/design_principles/neumorphism/`, `.skills/design_principles/swiss_grid/`, or `.skills/design_principles/neo_brutalism/`) and inspect the co-located `images/` specification sheets.
5. **Decision Matrix**: Use the guide's Decision Matrix to choose the right platform-native feature.
6. **Anti-Pattern Audit**: Check the guide's **Do Not** table to ensure no obsolete patterns (e.g. `!important`, physical coordinates, lodash imports, `forEach(async)`) are introduced.
```

If your project also uses [**`AGENTS.md`**](./AGENTS.md) (for OpenAI Codex, Cursor, or tools following that convention), ensure it points to `CLAUDE.md`:

```markdown
# AGENTS.md
Instructions for AI coding agents live in [CLAUDE.md](./CLAUDE.md). Please read and follow CLAUDE.md before making changes.
```

---

## Domains

### 🎨 [CSS Native Skills & Best Practices](./css/README.md)
Platform-native, zero-dependency modern CSS standards:
- **[Layout & Positioning](./css/css_layout_skill_guide.md)**: Flexbox min-size trap (`min-inline-size: 0`), CSS Grid RAM, Subgrid, Container Queries, Logical Properties, Viewport units.
- **[Architecture & Scoping](./css/css_architecture_skill_guide.md)**: Cascade Layers (`@layer`), Specificity zeroing (`:where()`), Native Scoping (`@scope`), Native Nesting, Minimal Modern Reset.
- **[Typography & Readability](./css/css_typography_skill_guide.md)**: Fluid typography (`clamp()`), `text-wrap: balance` & `pretty`, multi-line `line-clamp`, `overflow-wrap: break-word`, `ch` measure, Variable Fonts.
- **[Animations & Visual Effects](./css/css_animation_skill_guide.md)**: Scroll-driven animations, View Transitions API, `@starting-style`, Anchor Positioning, `linear()` springs.
- **[Colors, Spaces & Dynamic Theming](./css/css_colors_skill_guide.md)**: OKLCH, Relative Color Syntax, `color-mix()`, `light-dark()`, Forced Colors / High Contrast mode.
- **[Selectors & State Management](./css/css_selectors_and_state_skill_guide.md)**: Relational selector (`:has()`), `:user-valid`/`:user-invalid`, `:focus-visible`, `:focus-within`, floating labels.
- **[Accessibility & Preference Ergonomics](./css/css_accessibility_skill_guide.md)**: Zero-side-effect `.sr-only`, High-contrast focus, `prefers-reduced-motion`, `pointer: coarse` tap targets.
- **[Modern UI Components & Form Controls](./css/css_components_and_forms_skill_guide.md)**: Native `<dialog>`, Popover API, `<details name>` exclusive accordions, `field-sizing: content`, `accent-color`.

### ⚡ [JavaScript Native Skills & Best Practices](./javascript/README.md)
Platform-native, zero-dependency modern ECMAScript standards and Web APIs:
- **[Modern Standard Library](./javascript/js_standard_library_skill_guide.md)**: `structuredClone()`, `Object.groupBy()`, `toSorted()`, `toReversed()`, `toSpliced()`, `crypto.randomUUID()`, `Promise.withResolvers()`, `array.at(-1)`.
- **[Async & Concurrency Control](./javascript/js_async_and_concurrency_skill_guide.md)**: `for...of` vs `Promise.all` (`forEach` async trap), `Promise.allSettled()`, `AbortController`, `AbortSignal.timeout()`, `AbortSignal.any()`.
- **[DOM & Event Architecture](./javascript/js_dom_and_events_skill_guide.md)**: Declarative listener teardown via `{ signal }`, atomic `replaceChildren()`, `<template>` cloning, `event.target.closest()`, `IntersectionObserver`, `ResizeObserver`, `FormData`.
- **[Error Handling & Diagnostics](./javascript/js_error_handling_skill_guide.md)**: Error causation (`{ cause: err }`), custom `class extends Error` hierarchies, safe error normalization, global async error boundaries.

### 🧙 [Wizz.js Native Skills & Best Practices](./wizz/README.md)
Zero-dependency, compiler-guaranteed reactive framework standards:
- **[Component Architecture & Composition](./wizz/wizz_components_skill_guide.md)**: Single-root element rule, safe template expression subset, component imports, `<wizz:head>` metadata, and scoped `<wizz:style>`.
- **[Reactivity, State & Lifecycle](./wizz/wizz_reactivity_and_state_skill_guide.md)**: Microtask batching via `let`, read-only `export let` props, cross-tab reactive persistence via `persist()`, and `onMount`/`onDestroy`.
- **[Template Directives & Control Flow](./wizz/wizz_template_directives_skill_guide.md)**: Structural `{#if}` branching, `{#each}` iteration constraints, container event delegation, and precomputed list data.
- **[Architecture, Tooling & Server Deployment](./wizz/wizz_architecture_and_workflow_skill_guide.md)**: Project scaffolding (`wizz init`), live dev server (`wizz dev`), SSR production builds (`wizz build --adapter node`), and server API routes (`src/server/api/`).

### 💠 [Design Principles — Modern Neumorphism (Soft UI)](./design_principles/neumorphism/README.md)
Multimodal-ready design rules, OKLCH palettes, WCAG 2.2 accessibility safeguards, and visual component sheets in [`design_principles/neumorphism/images/`](./design_principles/neumorphism/images/):
- **[Foundations, Light Physics & Tokens](./design_principles/neumorphism/neumorphism_foundations_and_tokens_skill_guide.md)**: $135^\circ$ single light-source law, 4 surface states (Extruded, Inset, Convex, Concave), `1:2` blur math, elevation & radius tokens.
- **[Color Palettes, OKLCH & Dynamic Theming](./design_principles/neumorphism/neumorphism_color_palettes_and_theming_skill_guide.md)**: 4 curated palettes (Cool Porcelain, Warm Alabaster Sand, Sage Mist, Obsidian Slate Dark), Relative Color Syntax shadow derivation, `light-dark()`.
- **[Layout Architecture, Cards & Depth Hierarchy](./design_principles/neumorphism/neumorphism_layout_and_cards_skill_guide.md)**: Shadow bleed clearance formula, alternating depth polarity (Raised $\rightarrow$ Inset $\rightarrow$ Raised), `2px` etched seam dividers, 30% Soft-UI density cap.
- **[Tactile Buttons & Action Controls](./design_principles/neumorphism/neumorphism_button_skill_guide.md)**: 4-state press physics, circular icon buttons, high-contrast Primary Accent CTAs, segmented button groups.
- **[Forms, Text Inputs & Selection Controls](./design_principles/neumorphism/neumorphism_forms_and_inputs_skill_guide.md)**: Recessed concave inputs, composite search bars, `:user-valid`/`:user-invalid` glow rings, tactile checkboxes & radios.
- **[Sliders, Toggle Switches & Tactile Dials](./design_principles/neumorphism/neumorphism_sliders_and_toggles_skill_guide.md)**: Recessed range sliders with filled progress channels & raised thumbs, binary `role="switch"` toggles, vertical faders, rotary knobs.
- **[Loaders, Spinners & Progress Indicators](./design_principles/neumorphism/neumorphism_loader_and_progress_skill_guide.md)**: Stationary carved donut spinners with rotating conic arcs, percentage gauges, linear progress bars, concentric ripples.
- **[Navigation, Tabs & WCAG Accessibility Rules](./design_principles/neumorphism/neumorphism_navigation_and_accessibility_skill_guide.md)**: Segmented pill tablists, floating navbars, pagination strips, and the 5 mandatory WCAG 2.2 accessibility safeguards.

### 📐 [Design Principles — Swiss Grid (International Typographic Style)](./design_principles/swiss_grid/README.md)
Mathematical editorial architecture, exposed `1px` hairlines, Neo-Grotesque typographic scales, and visual specification sheets in [`design_principles/swiss_grid/images/`](./design_principles/swiss_grid/images/):
- **[Foundations, Exposed Architecture & Tokens](./design_principles/swiss_grid/swiss_grid_foundations_and_tokens_skill_guide.md)**: 12-column mathematical grid, seamless `gap: 1px` background-bleed hairlines, `2px–4px` structural rules, zero-radius law, coordinate markers.
- **[Typography, Scale Contrast & Editorial Hierarchy](./design_principles/swiss_grid/swiss_grid_typography_and_hierarchy_skill_guide.md)**: Neo-Grotesque + Technical Monospace pairing, tight `0.92` display leading, flush-left ragged-right rule, scholarly marginalia/sidenote rails, `4px` top-ruled pull quotes.
- **[Color Palettes, Signal Accents & Hard-Edge Inversion](./design_principles/swiss_grid/swiss_grid_color_palettes_and_theming_skill_guide.md)**: 90/10 monochrome + Single Signal Accent law, 4 canonical palettes (Zurich Archival Standard, Basel Newsprint, Geneva Concrete, Midnight Monolith Dark), full-cell hover polarity flips.
- **[Magazine Layouts, Asymmetric Spreads & Subgrid Cards](./design_principles/swiss_grid/swiss_grid_editorial_layout_and_cards_skill_guide.md)**: Asymmetric 8-col/4-col hero spreads, cross-aligned `subgrid` article feeds, zero-radius monochrome image plates.
- **[Zero-Radius Buttons, Cell Triggers & Action Bars](./design_principles/swiss_grid/swiss_grid_button_and_action_skill_guide.md)**: Hard-edge polarity-inverting buttons, Primary Signal Red CTAs, Full-Width Grid-Cell Action Bars, bracketed micro-links (`[ READ ESSAY → ]`), segmented filter strips.
- **[Architectural Forms, Inputs & Selection Controls](./design_principles/swiss_grid/swiss_grid_forms_and_inputs_skill_guide.md)**: `2px` bottom-rule inputs, numbered catalog labels, seamless cell-framed subscription forms, `:user-invalid` red rules, zero-radius square checkboxes & radios.
- **[Data Tables, Archive Indexes & KPI Matrices](./design_principles/swiss_grid/swiss_grid_tables_and_data_index_skill_guide.md)**: Full-row Signal Red hover-inversion archive directories, exposed `1px` hairline tables with `tabular-nums`, monolithic KPI callout cells.
- **[Mastheads, Sticky TOCs, Status Tickers & Accessibility](./design_principles/swiss_grid/swiss_grid_navigation_and_status_skill_guide.md)**: Cell-partitioned editorial mastheads, sticky numbered TOC rails, `scroll()` reading progress bars, step tickers, and WCAG 2.2 zero-radius focus rings.

### ⚡ [Design Principles — Neo-Brutalism (Neubrutalism)](./design_principles/neo_brutalism/README.md)
Exposed `2px–4px` ink borders, `0px`-blur hard offset shadows, coupled mechanical push-button physics, and visual specification sheets in [`design_principles/neo_brutalism/images/`](./design_principles/neo_brutalism/images/):
- **[Foundations, Hard Shadows & Push-Button Physics](./design_principles/neo_brutalism/neo_brutalism_foundations_and_tokens_skill_guide.md)**: Exposed stroke tiers (`2px`/`3px`/`4px`), zero-blur offset shadow tiers (`3px` to `12px`), and the Invariant Translation + Shadow Coupling Law ($\Delta\text{translate} + \Delta\text{shadow} = \text{constant}$).
- **[Color Palettes, OKLCH & Dark Mode Shadows](./design_principles/neo_brutalism/neo_brutalism_color_palettes_and_theming_skill_guide.md)**: Warm Cream (`#FAF6EE`) & Putty (`#EFECE6`) canvases, 4 canonical palettes (Gumroad Pop, Studio Pastel, Indie Terminal, Cyber Brutal Dark), and colored hard shadows in dark mode.
- **[Typography, Vinyl Stickers & Marquee Strips](./design_principles/neo_brutalism/neo_brutalism_typography_and_stickers_skill_guide.md)**: Heavy display grotesques + monospace specs, `-webkit-text-stroke` with `paint-order: stroke fill`, boxed inline highlights, tilted corner stickers, and accessible CSS marquees.
- **[Retro OS Windows, Color-Blocked Cards & Bento Grids](./design_principles/neo_brutalism/neo_brutalism_layout_cards_and_windows_skill_guide.md)**: Mac OS 8 / Win95 window chrome (`.nb-window`), compartmentalized pricing cards, overlapping corner badges, and shadow clearance grid gutters.
- **[Mechanical Push-Buttons, Keycaps & Segmented Bars](./design_principles/neo_brutalism/neo_brutalism_button_skill_guide.md)**: 3-state mechanical displacement (Idle `5px 5px 0` $\rightarrow$ Hover Lift `8px 8px 0` $\rightarrow$ Active Slam `0px 0px 0`), square icon keycaps, and connected segmented button groups.
- **[Chunky Forms, Inputs & Validation Badges](./design_principles/neo_brutalism/neo_brutalism_forms_and_inputs_skill_guide.md)**: `3px`-bordered inputs, warm yellow focus shifts (`#FFFDF0`) with pink hard shadows, attached input+submit bars, custom `28px` checkboxes/radios, and `:user-invalid` error stickers.
- **[Hardware Toggles, Blocky Sliders, Progress & Steppers](./design_principles/neo_brutalism/neo_brutalism_toggles_sliders_and_loaders_skill_guide.md)**: Mechanical rocker switches (`role="switch"`), square hard-shadowed range faders, retro 10-cell battery progress meters, hazard stripe bars, and numbered checkout steppers.
- **[Boxed Navbars, Manila-Folder Tabs, `<dialog>` Modals & WCAG 2.2](./design_principles/neo_brutalism/neo_brutalism_navigation_dialogs_and_accessibility_skill_guide.md)**: Compartmentalized top navbars, seam-overlapping folder tabs, native `<dialog>` OS modal windows, halftone backdrops, and `3px dashed` `:focus-visible` outlines.

---

## Agent Instructions & Authoring Guidelines

All instructions for AI agents operating in this repository itself live in [**`CLAUDE.md`**](./CLAUDE.md) (the single source of truth referenced by [`AGENTS.md`](./AGENTS.md)). Agents contributing to this repo must follow the standardized skill guide schema, decision matrices, and verification checklists outlined there.
