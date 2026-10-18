# JavaScript Native Testing, Benchmarking & Regression Skills

A zero-dependency, platform-native JavaScript verification knowledge base engineered specifically for AI coding agents. This directory provides production-grade decision matrices, statistical benchmarking harnesses, deterministic race-condition reproduction patterns, and strict anti-pattern rules for modern JavaScript runtimes (Node.js, V8, and modern browser engines).

---

## Mission & Purpose

When left to their default training weights, AI coding agents frequently introduce testing and benchmarking anti-patterns in JavaScript:
- Installing heavy legacy test toolchains (`jest`, `babel-jest`, `mocha`, `chai`, `sinon`) when Node.js has a fast, native ESM test runner (`node:test`) and strict assertion module (`node:assert/strict`) built in.
- Writing slow, flaky `setTimeout` sleeps instead of using scoped fake timers (`t.mock.timers`) or deterministic async barriers (`Promise.withResolvers()`).
- Producing invalid microbenchmarks using `Date.now()` that fail to warm up V8's TurboFan JIT compiler or get completely optimized away by Dead-Code Elimination (DCE).
- Fixing bugs without first proving a failing reproduction test (**Red-Before-Green**), or relying on giant, brittle `.snap` snapshot dumps that developers blindly update.

This directory equips AI agents with **authoritative, modern JavaScript testing and benchmarking standards** that pair directly with [`javascript/`](../../javascript/README.md), [`css/`](../../css/README.md), [`wizz/`](../../wizz/README.md), and [`testing/workflows/`](../workflows/README.md).

---

## Core Verification Principles

1. **Zero-Dependency Native Runner First**  
   Default to `node:test` (`node --test`) and `node:assert/strict` for unit and integration testing. Reserve browser automation (`@playwright/test`) for real DOM, CSS layout, and end-to-end verification.

2. **Scoped Lifecycle Isolation (`t.mock` & `t.after`)**  
   Bind all spies, method overrides, fake timers, and server teardowns to the test context `t` (`t.mock.method`, `t.mock.timers`, `t.after`, `t.signal`) so no test can ever pollute or hang subsequent tests.

3. **V8-Aware Statistical Benchmarking**  
   Never trust a single cold-loop average. Always run a JIT warmup phase, feed pure return values into an exported sink to prevent Dead-Code Elimination, interleave A/B comparison batches, and report `p50` (median), `p95`, `p99`, and heap allocation delta (`bytesPerOp`).

4. **Deterministic Concurrency & Red-Before-Green Regression**  
   Reproduce every bug with a failing test before patching production code. Test async race conditions deterministically with `Promise.withResolvers()` barriers and guard state immutability with `Object.freeze()` / `structuredClone()`.

5. **Behavior & Accessibility Over Implementation Details**  
   Query DOM elements by semantic role and accessible name (`getByRole`, `getByLabel`) and verify computed geometry invariants (`getComputedStyle`, `boundingBox`) rather than fragile CSS utility classes or giant HTML snapshots.

---

## Directory Index

| Guide | Description | Key Modern Features |
| :--- | :--- | :--- |
| **[`js_testing_standards_skill_guide.md`](./js_testing_standards_skill_guide.md)** | Unit, integration, and DOM testing standards | `node:test`, `node:assert/strict`, table-driven `await t.test()`, `t.mock.method()`, `t.mock.timers`, `t.signal` + `t.after()`, Playwright semantic locators (`getByRole`). |
| **[`js_benchmark_testing_skill_guide.md`](./js_benchmark_testing_skill_guide.md)** | Statistical micro/macro benchmarking & memory profiling | `performance.now()`, V8 warmup loops, DCE blackhole sinks, `p50`/`p95`/`p99`/`RME%`, interleaved A/B trials, `process.memoryUsage().heapUsed`, `PerformanceObserver`. |
| **[`js_regression_testing_skill_guide.md`](./js_regression_testing_skill_guide.md)** | Bug reproduction, concurrency races, and UI geometry regression | Red-Before-Green protocol, `Promise.withResolvers()` race barriers, `err.cause` chain assertions, `deepFreeze()` mutation guards, computed CSS & component screenshot invariants. |

---

## Agent Usage Instructions

When writing tests, benchmarks, or bug fixes in JavaScript:
1. **Select the Matching Guide**:
   - Writing new unit/integration/DOM tests $\rightarrow$ [`js_testing_standards_skill_guide.md`](./js_testing_standards_skill_guide.md).
   - Measuring speed, memory allocations, or comparing algorithms $\rightarrow$ [`js_benchmark_testing_skill_guide.md`](./js_benchmark_testing_skill_guide.md).
   - Fixing a bug, race condition, or visual/CSS layout defect $\rightarrow$ [`js_regression_testing_skill_guide.md`](./js_regression_testing_skill_guide.md).
2. **Execute the Verification Loop**:
   Pair these guides with [`../workflows/testing_and_review_loops_skill_guide.md`](../workflows/testing_and_review_loops_skill_guide.md) and [`../workflows/code_review_standards_skill_guide.md`](../workflows/code_review_standards_skill_guide.md) to run closed-loop test and review cycles before completing any task.
