# Engineering Workflows — Code Review Standards & Adversarial Auditing

## Code Review Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Classifying review findings | 4-Tier Severity Taxonomy (`BLOCKER`, `MAJOR`, `MINOR`, `NIT`) | Distinguishes ship-stopping correctness/security bugs from optional stylistic suggestions so authors know exactly what blocks merge. |
| Catching AI-hallucinated legacy code | Cross-referencing domain `Do Not` tables | Systematically eliminates `innerHTML`, `forEach(async)`, physical CSS properties (`margin-left`), `outline: none`, and missing error `cause`. |
| Auditing asynchronous & event code | Cancellation & Teardown Audit (`AbortSignal`, `onDestroy`) | Prevents silent memory leaks, orphaned timers, and out-of-order race conditions in long-lived sessions. |
| Auditing UI, CSS & Design Systems | WCAG 2.2 + Design System Invariant Audit | Verifies `:focus-visible`, `prefers-reduced-motion`, `forced-colors`, logical properties, and design token laws before visual sign-off. |
| Auditing test quality in a PR | Mutation Thought-Experiment ("Would this test fail if...") | Catches tautological tests, over-mocked tests, and missing boundary/error assertions that inflate coverage without verifying behavior. |
| Formatting review feedback | File + Line Range + Concrete Counterexample + Drop-In Fix | Eliminates vague commentary ("Consider improving error handling") in favor of deterministic, immediately actionable remediation. |

---

## Severity Taxonomy & Triage Contract

Every review finding must be tagged with an explicit severity level. Never mix blocking defects with minor style preferences in an undifferentiated bullet list:

| Severity | Merge Gate | Criteria |
| :--- | :--- | :--- |
| **`[BLOCKER]`** | **Must fix before completion** | Runtime crashes, data corruption, security vulnerabilities (XSS, injection, secret exposure), memory/listener leaks, broken accessibility (`outline: none`), or missing regression tests for a bug fix. |
| **`[MAJOR]`** | **Must fix or explicitly justify** | Violation of `agent-skills` `Do Not` rules, unhandled edge cases, missing `AbortSignal` cancellation, in-place mutation of reactive inputs, or $O(N^2)$ performance traps. |
| **`[MINOR]`** | **Recommended improvement** | Suboptimal readability, redundant state derivation, weak test assertion specificity (e.g. checking truthiness instead of exact structure). |
| **`[NIT]`** | **Optional / Non-blocking** | Naming polish, comment clarity, or minor organizational consistency. |

---

## Multi-Pass Adversarial Review Protocol

AI agents often perform superficial single-pass reviews ("The code looks clean and well-structured!"). Instead, execute a **4-pass adversarial audit** where you assume bugs exist and actively try to construct an input or timing sequence that breaks the diff:

### Pass 1: Correctness, Boundary & Concurrency Audit
Construct concrete counterexamples against the changed code:
1. **Empty & Extremal Inputs**: What happens on `[]`, `{}`, `""`, `0`, `NaN`, `null`, `undefined`, or Unicodeastral-plane strings (`"👨‍👩‍👧‍👦"`)?
2. **Async Interleaving & Cancellation**:
   - If the function is called twice rapidly, can call #1 resolve after call #2 and overwrite newer state?
   - Are `forEach(async () => ...)` loops hidden in the diff instead of `for...of` or `Promise.all()`?
   - Does every `fetch()`, stream, and `addEventListener()` accept or attach an `AbortSignal`?
3. **Immutability & Error Causation**:
   - Does the code mutate caller-owned arrays (`sort`, `splice`, `reverse`) instead of using `.toSorted()`, `.toSpliced()`, `.toReversed()`?
   - When catching and re-throwing errors, is `{ cause: err }` preserved?

```javascript
// ❌ [BLOCKER] Review Catch: Uncanceled listener + XSS sink + lost error cause
export function mountLiveFeed(container, endpoint) {
  window.addEventListener('online', async () => {
    try {
      const res = await fetch(endpoint);
      const data = await res.json();
      container.innerHTML = `<div class="feed">${data.title}</div>`;
    } catch (err) {
      throw new Error('Failed to refresh feed: ' + err.message);
    }
  });
}

// ✅ Remediated Replacement: Signal teardown + safe DOM construction + error causation
export function mountLiveFeed(container, endpoint, { signal } = {}) {
  window.addEventListener('online', async () => {
    try {
      const res = await fetch(endpoint, {
        signal: signal ? AbortSignal.any([signal, AbortSignal.timeout(8000)]) : AbortSignal.timeout(8000)
      });
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();

      const item = document.createElement('div');
      item.className = 'feed';
      item.textContent = data.title;
      container.replaceChildren(item);
    } catch (err) {
      if (err.name === 'AbortError') return;
      throw new Error('Failed to refresh feed', { cause: err });
    }
  }, { signal });
}
```

### Pass 2: CSS, Wizz.js & Design System Compliance Audit
When reviewing frontend components, stylesheets, or Wizz.js `.wizz` files, verify compliance against the repository's domain laws:
- **CSS Logical Properties**: Reject physical directional properties (`width`, `height`, `margin-left`, `padding-top`, `left`/`right`) in favor of flow-relative logical equivalents (`inline-size`, `block-size`, `margin-inline-start`, `padding-block-start`, `inset-inline`).
- **Accessibility Safeguards (WCAG 2.2)**:
  - Reject `outline: none` or `outline: 0` unless paired with an explicit high-contrast `:focus-visible` indicator.
  - Verify interactive tap targets meet minimum sizing (`44px × 44px` on coarse pointers) and honor `@media (prefers-reduced-motion: reduce)` and `@media (forced-colors: active)`.
- **Wizz.js Contracts**:
  - Verify single-root element architecture, no direct mutation of `export let` props, and cleanup of timers/listeners inside `onDestroy()`.
- **Design System Physics**:
  - **Neumorphism**: Single $135^\circ$ light source, alternating depth polarity, never nest extruded cards directly inside extruded cards without an inset well or seam.
  - **Swiss Grid**: Exposed `1px` hairlines (`gap: 1px`), `0px` border radius, flush-left ragged-right body measure (`60ch–70ch`).
  - **Neo-Brutalism**: `0px` shadow blur, coupled translation + shadow invariant ($\Delta\text{translate} + \Delta\text{shadow} = \text{constant}$).
  - **Claymorphism**: 3-layer `box-shadow` inflation (1 outer ambient + 2 opposing `inset` bevels), $R \ge 2 \times \text{offset}$ radius law.

### Pass 3: Security & Trust-Boundary Audit
- **DOM Injection**: Reject `innerHTML`, `outerHTML`, `insertAdjacentHTML`, and `document.write` with dynamic data.
- **Prototype & Object Injection**: Reject unvalidated dynamic key assignment (`target[userKey] = val`) without `Object.hasOwn()` or `Map`.
- **ReDoS & Resource Exhaustion**: Reject unbounded nested quantifiers in regular expressions and unpaginated/unbounded in-memory collection buffering.

### Pass 4: Test & Benchmark Sufficiency Audit
Perform the **Mutation Thought-Experiment** on every test file in the diff:
- *"If I delete the core state update or invert the `<` condition to `<=` in the implementation, will a test in this PR fail?"*
- Verify that bug fixes include a regression test covering the exact boundary input, and performance changes include a warmed, statistically valid benchmark comparison.

---

## Structured Review Output Schema

When reporting code review findings (either to a user or during an internal self-review loop), format findings deterministically:

```markdown
## Code Review Summary
- **Verdict**: `CHANGES_REQUESTED` (1 Blocker, 1 Major, 1 Minor)
- **Domains Audited**: `javascript/async`, `css/accessibility`, `testing/javascript`

### Findings

1. **`[BLOCKER]` Unbounded fetch and listener leak on component teardown**
   - **Location**: `src/components/SearchBox.js:24-38`
   - **Failure Mode**: Rapid input dispatches concurrent `fetch()` calls without aborting prior in-flight requests; if response #1 arrives after response #2, stale results overwrite current results.
   - **Fix**: Maintain an active `AbortController`, call `activeController.abort()` before starting a new request, and pass `{ signal }` to `fetch()`.

2. **`[MAJOR]` Physical CSS properties violate logical layout standard**
   - **Location**: `src/styles/card.css:12-19`
   - **Failure Mode**: `margin-left: 16px` and `width: 100%` break RTL localization and violate `css_layout_skill_guide.md`.
   - **Fix**: Replace with `margin-inline-start: 16px` and `inline-size: 100%`.
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| Rubber-stamping code with generic praise ("Looks good, clean implementation!") | Executing the 4-pass adversarial audit and constructing concrete boundary counterexamples | Polite summaries miss race conditions, XSS sinks, and subtle off-by-one regressions. |
| Leaving vague comments ("Maybe handle errors better here?") | Specifying severity, concrete failure trigger, and a drop-in code fix | Vague feedback forces extra round-trips and leaves room for misinterpretation. |
| Reviewing implementation files while ignoring the test files | Auditing tests with the Mutation Thought-Experiment (`node:assert/strict`, `t.mock`, boundary cases) | Weak or tautological tests create false confidence while letting regressions slip through. |
| Nitpicking variable names while missing a missing `AbortSignal` or `outline: none` | Prioritizing `[BLOCKER]` and `[MAJOR]` correctness, security, and accessibility defects first | High-severity functional and accessibility bugs must never be buried under stylistic noise. |
| Approving a bug fix PR that has zero new or updated regression tests | Requiring a Red-Before-Green regression test that fails without the fix | Untested bug fixes routinely regress during subsequent refactors. |
