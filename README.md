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
3. **Decision Matrix**: Use the guide's Decision Matrix to choose the right platform-native feature.
4. **Anti-Pattern Audit**: Check the guide's **Do Not** table to ensure no obsolete patterns (e.g. `!important`, physical coordinates, lodash imports, `forEach(async)`) are introduced.
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

---

## Agent Instructions & Authoring Guidelines

All instructions for AI agents operating in this repository itself live in [**`CLAUDE.md`**](./CLAUDE.md) (the single source of truth referenced by [`AGENTS.md`](./AGENTS.md)). Agents contributing to this repo must follow the standardized skill guide schema, decision matrices, and verification checklists outlined there.
