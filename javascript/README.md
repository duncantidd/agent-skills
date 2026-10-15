# JavaScript Native Skills & Best Practices

A zero-dependency, platform-native JavaScript knowledge base engineered specifically for AI coding agents. This directory provides production-ready decision matrices, modern code patterns, and explicit anti-pattern rules across modern ECMAScript standards and Web APIs.

---

## Mission & Purpose

Large language models frequently default to outdated JavaScript habits learned from historical training data:
- Reaching for heavy utility libraries (e.g. `lodash`, `uuid`, `rimraf`) when ECMAScript and Node.js have had native equivalents for years.
- Falling into asynchronous traps (e.g. `forEach(async () => ...)` running out-of-order, uncancelable `fetch` requests, unhandled rejections).
- Writing fragile, memory-leaking DOM operations (e.g. `element.innerHTML` XSS hazards, dangling event listeners without `AbortSignal`).
- Destroying call stacks during error re-throwing instead of using native Error Causation (`{ cause: err }`).

This directory equips AI agents with **authoritative, modern, platform-native JavaScript standards** to produce clean, high-performance, secure, and dependency-light code.

---

## Core Engineering Principles

1. **Zero-Dependency Native First**  
   Favor native ECMAScript and Web standard APIs (`structuredClone`, `Object.groupBy`, `crypto.randomUUID()`, `Promise.withResolvers`) before installing third-party npm packages.

2. **Immutable Data Operations**  
   Avoid mutating arrays or collections in-place. Use native non-mutating array operations (`toSorted()`, `toReversed()`, `toSpliced()`, `with()`) to prevent side effects in reactive UI frameworks and data pipelines.

3. **Signal-Driven Async & Cancellation**  
   Every asynchronous operation must provide a cancellation or timeout path using `AbortController`, `AbortSignal.timeout()`, and `AbortSignal.any()`. Avoid uncancelable async loops and the `forEach` async trap.

4. **Defense-in-Depth DOM Safety**  
   Never inject untrusted markup using `innerHTML`. Use atomic child updates (`replaceChildren`), `<template>` cloning, and declarative event teardown via `{ signal }`.

5. **Diagnostic Error Causation**  
   Never concatenate error strings. Use `new Error("Message", { cause: originalError })` to preserve the complete underlying call stack across service and API boundaries.

---

## Directory Index

| Guide | Description | Key Modern Features |
| :--- | :--- | :--- |
| **[`js_standard_library_skill_guide.md`](./js_standard_library_skill_guide.md)** | Modern ECMAScript utilities & zero-dependency collection handling | `structuredClone()`, `Object.groupBy()`, `Map.groupBy()`, `toSorted()`, `toReversed()`, `toSpliced()`, `crypto.randomUUID()`, `Promise.withResolvers()`, `array.at(-1)`. |
| **[`js_async_and_concurrency_skill_guide.md`](./js_async_and_concurrency_skill_guide.md)** | Robust asynchronous execution, cancellation, and streaming | `for...of` vs `Promise.all` (`forEach` async trap), `Promise.allSettled()`, `AbortController`, `AbortSignal.timeout()`, `AbortSignal.any()`, readable streams. |
| **[`js_dom_and_events_skill_guide.md`](./js_dom_and_events_skill_guide.md)** | Modern DOM manipulation, event lifecycle, and observers | Declarative listener teardown via `{ signal }`, atomic `replaceChildren()`, `<template>` cloning, `event.target.closest()`, `IntersectionObserver`, `ResizeObserver`, `FormData`. |
| **[`js_error_handling_skill_guide.md`](./js_error_handling_skill_guide.md)** | Resilient error management and stack preservation | Error causation (`{ cause: err }`), custom `class extends Error` hierarchies, safe error normalizer, global async error boundaries. |

---

## Agent Usage Instructions

When generating or refactoring JavaScript:
1. **Identify the Task Domain**: Match the programming task to the relevant guide above.
2. **Consult the Decision Matrix**: Check the guide's Decision Matrix table to choose the right platform-native API.
3. **Verify Anti-Patterns**: Check the guide's **Do Not** table to ensure obsolete patterns (e.g. `lodash` imports, `forEach(async)`, `innerHTML`, lost error causes) are not introduced.
4. **Enforce Immutability & Cancellation**: Default to non-mutating array methods and signal-driven async cancellation.
