# CLAUDE.md — Agent Operating Instructions & Repository Guidelines

This document is the **single source of truth** for AI coding agents operating within this repository. All AI agents (Claude, OpenAI Codex, Antigravity, Gemini, etc.) must read, follow, and uphold the instructions defined here before creating, editing, or evaluating any files.

---

## 1. Repository Mission

The `agent-skills` repository is a curated collection of production-grade skill guides, design patterns, decision matrices, and anti-pattern rules. 

Its explicit goal is to **serve as an external memory and authoritative knowledge base for AI coding agents**, preventing them from generating obsolete legacy code (e.g. 2010-era CSS hacks, unnecessary preprocessor bloat, fragile DOM manipulations) and guiding them toward modern, accessible, platform-native standards.

---

## 2. Directory Architecture & Conventions

The repository is organized by technology domain:

```text
agent-skills/
├── CLAUDE.md                    # Core operating instructions for all agents (this file)
├── AGENTS.md                    # Pointer file for agent systems requiring this convention
├── README.md                    # Repository-wide introduction and domain index
│
├── css/                         # CSS Native domain knowledge base
│   ├── README.md                # Domain mission, philosophy, index, and retrieval guide
│   ├── css_layout_skill_guide.md
│   ├── css_architecture_skill_guide.md
│   ├── css_typography_skill_guide.md
│   ├── css_animation_skill_guide.md
│   ├── css_colors_skill_guide.md
│   ├── css_selectors_and_state_skill_guide.md
│   ├── css_accessibility_skill_guide.md
│   └── css_components_and_forms_skill_guide.md
│
├── javascript/                  # JavaScript Native domain knowledge base
│   ├── README.md                # Domain mission, principles, and index
│   ├── js_standard_library_skill_guide.md
│   ├── js_async_and_concurrency_skill_guide.md
│   ├── js_dom_and_events_skill_guide.md
│   └── js_error_handling_skill_guide.md
│
├── wizz/                        # Wizz.js Native domain knowledge base
│   ├── README.md                # Domain mission, contracts, and index
│   ├── wizz_components_skill_guide.md
│   ├── wizz_reactivity_and_state_skill_guide.md
│   ├── wizz_template_directives_skill_guide.md
│   └── wizz_architecture_and_workflow_skill_guide.md
│
├── design_principles/           # Visual design systems & multimodal UI packages
│   ├── README.md                # Design systems index
│   └── neumorphism/             # Modern Neumorphism (Soft UI) design package
│       ├── README.md            # Neumorphism laws, index, and multimodal guide
│       ├── images/              # Component & palette reference sheets for agents
│       ├── neumorphism_foundations_and_tokens_skill_guide.md
│       ├── neumorphism_color_palettes_and_theming_skill_guide.md
│       ├── neumorphism_layout_and_cards_skill_guide.md
│       ├── neumorphism_button_skill_guide.md
│       ├── neumorphism_forms_and_inputs_skill_guide.md
│       ├── neumorphism_sliders_and_toggles_skill_guide.md
│       ├── neumorphism_loader_and_progress_skill_guide.md
│       └── neumorphism_navigation_and_accessibility_skill_guide.md
│
└── [future-domains]/            # (e.g., typescript/, web-perf/, etc.)
    ├── README.md
    └── [domain]_[topic]_skill_guide.md
```

### File Naming Convention
- Domain directory: lowercase single word (e.g., `css`, `html`, `perf`) or nested under `design_principles/<system>/` for visual design packages.
- Skill guide filename: `[domain]_[topic]_skill_guide.md` in lowercase `snake_case` (e.g., `css_layout_skill_guide.md`, `neumorphism_button_skill_guide.md`).
- Visual reference assets (for design packages): `design_principles/<system>/images/` referenced via relative links (`./images/...`).
- Domain overview: `README.md` in the root of the domain directory.

---

## 3. Mandatory Structure for Skill Guides

To ensure deterministic parsing, retrieval, and consumption by AI agents, **every skill guide must strictly adhere to the following schema**:

```markdown
# [Domain] Native — [Topic Title]

## [Topic] Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| [Specific engineering goal] | [Platform tool or API] | [Clear, technical justification] |

---

## [Key Feature 1 Heading]
[Concise explanation of the modern approach and why it replaces older paradigms.]

```[language]
/* Canonical, clean, production-ready code example */
/* Well-commented highlighting modern nuance */
```

---

## [Key Feature 2 Heading]
...

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| [Legacy or buggy syntax] | [Modern replacement] | [Concrete failure mode or performance penalty] |
```

### Guidelines for Each Section:
1. **Title**: Prefixed with `# [Domain] Native — [Topic]` (or `# [Domain] — [Topic]`).
2. **Decision Matrix**: High-density table directly beneath the title. Answers the question: *"I need to accomplish X, which tool should I select and why?"*
3. **Core Sections**:
   - Zero conversational fluff. Maximum information density.
   - Clear, canonical code snippets with helpful comments.
   - Address edge cases, progressive enhancement (`@supports`), and responsive nuances.
4. **"Do Not" Anti-Pattern Table**:
   - Mandatory final section.
   - Explicitly catalogs obsolete patterns, traps, and anti-patterns that AI agents frequently hallucinate or generate from outdated training data.

---

## 4. How AI Agents Must Consume These Skills

When an AI agent is tasked with writing or refactoring code in a user project:

1. **Check the Domain Index First**:
   Read the corresponding domain `README.md` (e.g. `css/README.md`) to discover which guides apply to the user's requirements.
2. **Query the Decision Matrix**:
   Use the guide's Decision Matrix to select the modern platform tool rather than defaulting to an outdated pattern or third-party library.
3. **Audit Against the "Do Not" Table**:
   Before outputting generated code, verify that none of the bad patterns listed in the guide's "Do Not" table are present.
4. **Default to Modern Standards**:
   - **Layout**: Always prefer logical properties (`inline-size`, `margin-inline`, `padding-block`, `inset`) over physical properties (`width`, `margin-left`, `top`).
   - **Sizing**: Use `clamp()` for fluid scaling and `min-block-size: 100dvh` for full-height viewports.
   - **Colors**: Prefer OKLCH and `light-dark()` over rigid Hex/HSL and duplicated `@media (prefers-color-scheme)` blocks.
   - **Accessibility**: Never output `outline: none` without `:focus-visible`, always provide accessible text for icons, and honor `@media (prefers-reduced-motion)`.

---

## 5. Authoring & Contribution Rules for Agents

When creating new skills or updating existing files in this repository:

1. **Platform Native Priority**:
   Only document standard, platform-native web features or established ecosystem best practices. Avoid vendor lock-in or fragile proprietary abstractions unless writing a library-vs-native comparison.
2. **Exhaustive Code Correctness**:
   All CSS and code snippets must be syntactically valid and tested against modern browser specifications (CSS Color 4/5, Selectors 4, CSS Grid 2/3, Cascade Layers 1, Popover API).
3. **Synchronize Domain README**:
   Whenever a skill guide is added, renamed, or significantly updated, the domain `README.md` table of contents **must be updated immediately** in the same task.
4. **Maintain Conciseness & Precision**:
   Do not add subjective opinions or verbose background essays. Prioritize code, tables, and concise rationales.
5. **Preserve Documentation Integrity**:
   Do not delete existing guidelines or docstrings unless they are demonstrably incorrect or obsolete under modern specifications.

---

## 6. Pre-Commit / Completion Checklist

Before finalizing any changes in this repository, run through this verification checklist:

- [ ] Does the new or edited file follow the mandatory skill guide schema?
- [ ] Is there an up-to-date Decision Matrix (`Goal | Tool | Why`)?
- [ ] Is there an up-to-date Do Not table (`BAD | GOOD | Why`)?
- [ ] Are all code snippets syntactically valid and commented?
- [ ] Are flow-relative logical properties used throughout?
- [ ] Is the file linked and described in the domain `README.md`?
- [ ] Is the root `README.md` updated if a new domain was introduced?
- [ ] Are there no uncommitted temporary scratch files left behind?
