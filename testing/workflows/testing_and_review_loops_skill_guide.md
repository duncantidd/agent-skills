# Engineering Workflows — Agent Testing & Review Loops

## Testing & Review Loops Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Building a new feature or module | **Loop 1**: Spec $\rightarrow$ Test $\rightarrow$ Implement $\rightarrow$ Adversarial Self-Review | Forces clear public contracts and edge-case coverage before implementation biases test design. |
| Fixing a reported bug or regression | **Loop 2**: Reproduce (**Red**) $\rightarrow$ Patch (**Green**) $\rightarrow$ Boundary Expansion | Proves root-cause diagnosis by observing the exact failure before changing a single line of production code. |
| Optimizing a slow function or UI path | **Loop 3**: Baseline Benchmark $\rightarrow$ Profile $\rightarrow$ Optimize $\rightarrow$ A/B Verify | Prevents premature or counter-productive micro-optimizations by requiring statistically significant p50/p99 and heap improvements. |
| Eliminating a flaky test | **Loop 4**: Isolate $\rightarrow$ Deterministic Barrier / Fake Clock $\rightarrow$ Stress Repeat (`--test-concurrency`) | Replaces arbitrary `setTimeout` retries with deterministic synchronization (`t.mock.timers`, `Promise.withResolvers`). |
| Finalizing any coding task | **Loop 5**: Pre-Completion Quality Gate & Anti-Pattern Audit | Ensures zero uncommitted scratch files, 100% passing tests, and zero violations of repository `Do Not` tables before reporting completion. |

---

## Architecture of Closed-Loop Agent Execution

An AI coding agent must never operate in a "fire-and-forget" linear pass (`Read Prompt -> Write Code -> Done`). Production-grade engineering requires **closed feedback loops** where every code change is challenged by automated execution and an adversarial self-review before completion.

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                    CLOSED-LOOP AGENT EXECUTION CYCLE                    │
│                                                                         │
│  [1. Contract & Edge-Case Matrix]                                       │
│                 │                                                       │
│                 ▼                                                       │
│  [2. Write/Update Failing Test (RED)] ◄───┐                             │
│                 │                         │ (New edge case discovered)  │
│                 ▼                         │                             │
│  [3. Minimal Clean Implementation]        │                             │
│                 │                         │                             │
│                 ▼                         │                             │
│  [4. Run Tests & Benchmarks (GREEN)] ─────┼───► (If FAIL: Diagnose &    │
│                 │                         │      fix root cause)        │
│                 ▼                         │                             │
│  [5. 4-Pass Adversarial Self-Review] ─────┘                             │
│                 │                                                       │
│                 ▼ (Zero BLOCKER / MAJOR findings)                       │
│  [6. Pre-Completion Verification Gate]                                  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Loop 1: Feature Implementation Loop (TDD + Self-Review)

When implementing a new feature, component, or API endpoint:

1. **Define the Contract & Edge-Case Matrix**:
   List the happy path, boundary conditions (`0`, empty collection, `null`/`undefined`), cancellation path (`AbortSignal`), and error causation contract (`{ cause: err }`).
2. **Write the Test Suite First (`Red`)**:
   Create the test file using `node:test` and `node:assert/strict` (or Playwright for UI) and execute it to confirm it fails for the expected reason.
3. **Implement Using Platform-Native Standards (`Green`)**:
   Write the implementation adhering to the relevant `agent-skills` domain guides (`javascript/`, `css/`, `wizz/`, `design_principles/`).
4. **Execute & Verify Coverage of Branches**:
   ```bash
   # Run Node's built-in test runner with experimental coverage enabled
   node --test --experimental-test-coverage
   ```
5. **Run the Adversarial Self-Review Pass**:
   Switch hats from *Author* to *Adversarial Reviewer* (following [`code_review_standards_skill_guide.md`](./code_review_standards_skill_guide.md)). Inspect your own `git diff` against the domain `Do Not` tables. If any `[BLOCKER]` or `[MAJOR]` issue is found, add a test case that exposes it and re-enter Step 2.

---

## Loop 2: Bugfix & Regression Loop (Strict Red-Before-Green)

When fixing a bug, crash, race condition, or CSS layout defect, **never edit the production file first**:

```javascript
// STEP 1 (RED): Write a minimal reproduction test in tests/regression.test.js
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { parseQueryParams } from '../src/url-parser.js';

test('regression: parseQueryParams() preserves repeated keys as an array without dropping empty values', () => {
  const input = '?tag=js&tag=&tag=css';
  const result = parseQueryParams(input);

  // Run `node --test` -> Confirm this FAILS with current implementation
  assert.deepEqual(result, { tag: ['js', '', 'css'] });
});
```

1. **Reproduce (`Red`)**: Run `node --test` and verify the new test fails with the exact symptom reported. If the test passes before your fix, your reproduction is incomplete—refine the test until it fails.
2. **Patch (`Green`)**: Apply the minimal root-cause fix in the production module (never a symptom-masking `try/catch` bandage).
3. **Expand the Boundary Neighborhood**: Ask: *"Where else in this module or sibling modules does this same assumption exist?"* Add parameterized subtests covering adjacent edge cases (e.g., encoded characters, missing `=`, trailing `&`).
4. **Full Suite Regression Check**: Run the entire test suite to verify zero collateral regressions.

---

## Loop 3: Performance & Benchmark Optimization Loop

Never claim a code change "improves performance" without running a closed-loop A/B benchmark (following [`../javascript/js_benchmark_testing_skill_guide.md`](../javascript/js_benchmark_testing_skill_guide.md)):

1. **Capture Baseline Metrics**:
   Write an interleaved benchmark comparing the current implementation against itself (sanity check: speedup $\approx 1.00\times$) and record baseline `p50`, `p95`, `p99`, and `bytesPerOp` heap allocation.
2. **Apply Targeted Optimization**:
   Replace high-overhead patterns (e.g., replacing $O(N^2)$ `array.includes` inside a loop with a `Set`, or replacing repeated DOM reflows with a single `DocumentFragment` / `replaceChildren()` batch).
3. **Verify correctness first, then benchmark**:
   - Run the unit/regression test suite (`node --test`) to confirm zero behavioral regressions.
   - Run the interleaved A/B benchmark (`node --expose-gc benchmarks/target.bench.js`).
4. **Evaluate the Stop/Revert Gate**:
   - If `p50` or `p99` did not improve beyond the Relative Margin of Error (`RME %`), or if heap allocation (`bytesPerOp`) spiked unreasonably, **revert the change**—prefer simpler, readable code over zero-gain complexity.

---

## Loop 4: Flaky Test Isolation & Elimination Loop

When a test fails intermittently, **never add `.retry(3)` or increase `setTimeout` durations**. Flakiness is a deterministic bug in either the test harness or the application's concurrency model:

1. **Identify the Non-Determinism Source**:
   - **Wall-clock timers**: Replace `setTimeout` waits with `t.mock.timers.enable()` and `t.mock.timers.tick()`.
   - **Async race order**: Replace uncontrolled concurrent promises with `Promise.withResolvers()` barriers.
   - **Shared mutable state**: Move module-level fixtures inside the `test()` block or deep-clone via `structuredClone()`.
   - **Port/resource collisions**: Bind test servers to port `0` (`server.listen(0)`) and register `t.after(() => server.close())`.
2. **Stress-Verify Under Concurrency**:
   Run the isolated test file repeatedly in parallel to prove the flake is eliminated:
   ```bash
   # Execute 50 concurrent runs to verify deterministic isolation
   node -e '
     import { run } from "node:test";
     import assert from "node:assert/strict";
     for (let i = 0; i < 20; i++) {
       const stream = run({ files: ["./test/search-race.test.js"], concurrency: true });
       stream.on("test:fail", (f) => { console.error(f); process.exit(1); });
     }
   '
   ```

---

## Loop 5: Pre-Completion Self-Review & Quality Gate

Before concluding any implementation or refactoring turn, execute this mandatory 5-step verification gate:

```markdown
### Agent Pre-Completion Verification Checklist
1. [ ] **Execution Proof**: Have all new and existing tests been executed (`node --test`) and passed with zero errors?
2. [ ] **Red-Before-Green Proof**: For bug fixes, was the regression test observed failing before the fix and passing after?
3. [ ] **Adversarial Diff Audit**: Has `git diff` been audited against the relevant domain `Do Not` tables (`javascript`, `css`, `wizz`, `design_principles`, `testing`) with zero `[BLOCKER]` or `[MAJOR]` violations?
4. [ ] **Resource & Cleanup Hygiene**: Are all temporary scratch scripts, debug `console.log` statements, and untracked test artifacts removed (`git status`)?
5. [ ] **Documentation Sync**: Are public API signatures, comments, and `README.md` indexes synchronized with the code changes?
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| Writing code and declaring the task complete without running tests | Executing `node --test` and verifying exit code `0` inside the loop | Untested AI-generated code frequently contains subtle import, syntax, or edge-case bugs. |
| Modifying a failing test's expected value just to make the suite green | Investigating whether the code introduced a regression or the spec legitimately changed | Weakening assertions to silence failures hides real production bugs. |
| Fixing a failing test by adding `await new Promise(r => setTimeout(r, 500))` | Synchronizing on explicit state transitions, events, or `Promise.withResolvers()` | Arbitrary sleeps mask race conditions and collapse under CI load. |
| Performing self-review in the same mental pass as authoring | Auditing the final `git diff` using the 4-Pass Adversarial Review checklist | Authors naturally overlook their own assumptions unless forced through an explicit counterexample checklist. |
| Leaving temporary benchmark or reproduction scripts untracked in the repo root | Turning temporary repros into permanent `node:test` regression tests and cleaning up scratch files | Pollutes the repository and loses the regression check for future contributors. |
