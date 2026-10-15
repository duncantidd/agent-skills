# Wizz.js Native Skills & Best Practices

A zero-dependency, platform-native knowledge base engineered specifically for AI coding agents developing applications with **Wizz.js**. This directory provides production-grade decision matrices, component patterns, compiler contracts, and explicit anti-pattern rules.

---

## Mission & Purpose

Wizz.js is built on a strict zero-dependency architectural mandate: no bloated preprocessors, no heavyweight runtime Virtual DOMs, and no external npm bundle chains. It gives developers and agents direct, unmediated access to modern Web Platform primitives, sub-millisecond layout updates, and compiler-guaranteed scoped styles.

Because AI coding agents frequently hallucinate patterns from React, Vue, or Svelte (e.g. arbitrary JS in templates, nested components inside iteration blocks, or client-side synthetic events), this directory serves as the **authoritative reference** for generating syntactically valid and performant Wizz applications.

---

## Core Engineering Principles

1. **Zero-Dependency Platform First**  
   Wizz applications compile directly to platform-native JavaScript and CSS. Rely on native browser APIs, modern ECMAScript standards, and system fonts rather than reaching for third-party npm libraries.

2. **Single-Root Component Contract**  
   Every `.wizz` file must contain exactly one native root element. Templates, optional `<script>`, `<wizz:style>`, and `<wizz:head>` blocks are statically validated at compile time.

3. **Deterministic Expression Boundaries**  
   Template interpolations `{...}` support only a safe subset of JavaScript (identifiers, member chains, literals, and strict equality). All data transformations, filters, and formatting must be precomputed in `<script>`.

4. **Event Delegation for Iterative Collections**  
   `{#each}` loop bodies forbid components, `on:` directives, and nested blocks. Handle dynamic list interactions via container event delegation using `closest('[data-id]')`.

5. **Integrated SSR and Server Routes**  
   Server routes live under `src/server/api/`, execute in a frozen request context, and deploy cleanly via `node dist/server.mjs` without separate backend frameworks.

---

## Directory Index

| Guide | Description | Key Modern Features |
| :--- | :--- | :--- |
| **[`wizz_components_skill_guide.md`](./wizz_components_skill_guide.md)** | Component structure, template expressions, and scoping | Single root element rule, safe expression subset, default imports, `<wizz:head>` metadata, and scoped `<wizz:style>`. |
| **[`wizz_reactivity_and_state_skill_guide.md`](./wizz_reactivity_and_state_skill_guide.md)** | Reactive state, props, persistence, and lifecycle | Microtask batching with `let`, read-only `export let` props, cross-tab reactive `persist()`, and `onMount`/`onDestroy`. |
| **[`wizz_template_directives_skill_guide.md`](./wizz_template_directives_skill_guide.md)** | Structural control flow, list iteration, and event delegation | `{#if ...} {:else}` branching, `{#each}` compiler constraints, native container event delegation, and precomputed list data. |
| **[`wizz_architecture_and_workflow_skill_guide.md`](./wizz_architecture_and_workflow_skill_guide.md)** | Project layout, CLI workflows, and production server | `wizz dev`, `wizz build --adapter node`, `src/server/api/` handlers, `.env.server` security, and diagnostic codes. |

---

## Agent Usage Instructions

When generating, editing, or refactoring Wizz.js code:
1. **Identify the Architecture Layer**: Match the task to the corresponding guide above.
2. **Consult the Decision Matrix**: Check the guide's Decision Matrix to select the idiomatic Wizz construct.
3. **Verify Compiler Constraints**:
   - Verify that all `.wizz` files have exactly one native root element.
   - Verify that template expressions `{...}` contain no function calls, ternaries, or arithmetic.
   - Verify that no components or `on:` directives are placed inside `{#each}` blocks.
   - Verify that props (`export let`) are never mutated in the child component.
4. **Audit Anti-Patterns**: Compare the generated code against the guide's **Do Not** table before completion.

---

## Project Integration & Router Configuration

When using `agent-skills` inside a Wizz.js project (typically vendored or linked as a submodule in `.skills/`), add the following router directive to your Wizz application's root `CLAUDE.md`.

This ensures agents respect the framework boundary without conflicts:

```markdown
## Domain Skills & Architectural Standards

Before authoring, refactoring, or reviewing code in this project, you MUST consult `.skills/`:

1. **Wizz Components & Architecture**:
   Consult `.skills/wizz/README.md`. Wizz compiler rules take precedence for `.wizz` file contracts, single-root templates, expression boundaries, reactive `let` state, and directives.
2. **JavaScript Logic & Server APIs**:
   Consult `.skills/javascript/README.md` for standard library usage, non-mutating array operations (`toSorted()`, `toReversed()`), async cancellation (`AbortSignal`), and error causation (`{ cause }`).
3. **Styling & Responsive Layout**:
   Consult `.skills/css/README.md` for flow-relative logical properties, modern layout (Grid/Flexbox/Container queries), and accessibility.
4. **Anti-Pattern Audit**:
   Always check the relevant guide's **Do Not** table to eliminate obsolete patterns before finalizing code.
```

### Precedence & Synergy Summary

| Layer | Primary Guide | Secondary Guide | Rationale |
| :--- | :--- | :--- | :--- |
| **`.wizz` Templates & Directives** | `wizz/` | — | Wizz compiler enforces template syntax, safe expressions, single-root elements, and block constraints. |
| **Component `<script>` Blocks** | `wizz/` | `javascript/` | Wizz defines `let` reactivity and `onMount`/`onDestroy`; JS guides provide non-mutating array operations and `AbortSignal` cleanup. |
| **Server Handlers (`src/server/api/`)** | `javascript/` | `wizz/` | Wizz defines frozen context and routing; JS guides define standard async streams and error causation. |
| **Styles (`App.css` & `<wizz:style>`)** | `css/` | `wizz/` | CSS guides provide modern logical properties and layout; Wizz handles compiler scoping via `[data-wizz-s]`. |
