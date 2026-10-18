# JavaScript Native — Regression Testing & Invariant Verification

## Regression Testing Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Locking in a bug fix | Red-Before-Green minimal reproduction test | Proves the test actually catches the bug before applying the fix, preventing false-positive "always passing" tests. |
| Reproducing async race conditions | `Promise.withResolvers()` barrier latches | Replaces flaky `setTimeout` race guesses with deterministic step-by-step interleaving of concurrent tasks. |
| Verifying error handling regressions | `assert.rejects()` + `err.cause` chain inspection | Ensures re-thrown errors never regress into swallowed stack traces or string-concatenated messages. |
| Detecting unintended state mutation | `Object.freeze()` / `structuredClone()` pre-snapshot | Immediately catches accidental in-place mutation of caller arguments or shared reactive state. |
| CSS & Design System layout regression | Playwright `getComputedStyle()` + Bounding-Box invariants | Verifies structural laws (e.g. `0px` border-radius in Swiss Grid, coupled shadow translation in Neo-Brutalism) without brittle full-HTML string snapshots. |
| Visual pixel regression | Playwright `expect(locator).toHaveScreenshot()` | Scoped to a specific component locator with animations disabled (`animations: 'disabled'`), avoiding full-page noise. |
| API / Serialization contract stability | Explicit structural schema assertions | Fails loudly only when public contract fields or types break, unlike 2,000-line `.snap` files that developers blindly regenerate. |

---

## The "Red-Before-Green" Bug Reproduction Protocol

Never patch production code before writing an automated regression test that reproduces the exact failure mode. Every regression test must document the failure trigger, execute the exact boundary input, and assert the invariant:

```javascript
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { paginateCursor } from '../src/pagination.js';

// Regression test naming convention: describe the invariant and the edge case trigger
test('regression: paginateCursor() does not drop last item when total equals pageSize', () => {
  const records = [
    { id: 'rec_1', createdAt: '2026-01-01T00:00:00Z' },
    { id: 'rec_2', createdAt: '2026-01-02T00:00:00Z' },
    { id: 'rec_3', createdAt: '2026-01-03T00:00:00Z' }
  ];

  // Snapshot original input to verify zero in-place mutation
  const snapshotBefore = structuredClone(records);

  const page = paginateCursor(records, { limit: 3, cursor: null });

  assert.equal(page.items.length, 3);
  assert.equal(page.items.at(-1).id, 'rec_3');
  assert.equal(page.nextCursor, null);
  assert.deepEqual(records, snapshotBefore, 'paginateCursor must not mutate input array');
});
```

---

## Deterministic Concurrency & Race-Condition Regression Tests

Async race conditions (e.g., out-of-order network responses overwriting newer UI state, or double-submission hazards) cannot be reliably tested with `setTimeout(50)` vs `setTimeout(10)`. Use `Promise.withResolvers()` as deterministic **execution barriers** to force exact interleavings:

```javascript
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { createSearchController } from '../src/search-controller.js';

test('regression: stale slow response from query #1 never overwrites newer query #2', async () => {
  // Create deterministic manual barriers for two concurrent network requests
  const request1 = Promise.withResolvers();
  const request2 = Promise.withResolvers();

  const queries = [];
  const fakeFetcher = (query, { signal }) => {
    queries.push({ query, signal });
    return query === 're' ? request1.promise : request2.promise;
  };

  const controller = createSearchController({ fetcher: fakeFetcher });

  // 1. User types 're' (Query #1 starts)
  const p1 = controller.search('re');

  // 2. User immediately types 'react' (Query #2 starts; Query #1 should be aborted)
  const p2 = controller.search('react');
  assert.equal(queries[0].signal.aborted, true, 'Stale request #1 must be aborted via AbortSignal');

  // 3. Query #2 resolves FIRST
  request2.resolve(['react-dom', 'react-router']);
  await p2;
  assert.deepEqual(controller.getResults(), ['react-dom', 'react-router']);

  // 4. Stale Query #1 resolves SECOND (late arrival over network)
  request1.resolve(['redux', 'relay']);
  await p1.catch(() => {}); // Ignore expected abort rejection

  // Invariant: UI state must still reflect Query #2, never overwritten by stale Query #1
  assert.deepEqual(controller.getResults(), ['react-dom', 'react-router']);
});
```

---

## Error Causation & Diagnostic Stack Regression

When refactoring service layers, verify that wrapped errors retain their `cause` property (`new Error(msg, { cause: err })`) and custom domain error classes:

```javascript
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { PaymentGatewayError, chargeCustomer } from '../src/payments.js';

test('regression: chargeCustomer() preserves underlying network error as err.cause', async () => {
  const rootSocketError = new TypeError('fetch failed: ECONNRESET');
  const failingTransport = async () => {
    throw rootSocketError;
  };

  await assert.rejects(
    () => chargeCustomer({ amountCents: 2500, transport: failingTransport }),
    (err) => {
      assert.equal(err instanceof PaymentGatewayError, true);
      assert.equal(err.code, 'PAYMENT_UPSTREAM_UNAVAILABLE');
      // Critical regression check: root cause must remain attached for observability
      assert.equal(err.cause, rootSocketError);
      return true;
    }
  );
});
```

---

## Immutability & Reactive State Regression (`Object.freeze`)

In reactive frameworks (such as Wizz.js, React, or signals-based stores), in-place mutation (`array.sort()`, `array.push()`, property mutation on props) silently breaks dirty-checking and change detection. Deep-freeze test inputs to guarantee pure functions never regress into mutating state:

```javascript
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { sortAndFilterTasks } from '../src/task-utils.js';

function deepFreeze(obj) {
  if (obj && typeof obj === 'object') {
    Object.freeze(obj);
    for (const value of Object.values(obj)) {
      deepFreeze(value);
    }
  }
  return obj;
}

test('regression: sortAndFilterTasks() never mutates frozen input state', () => {
  const rawTasks = deepFreeze([
    { id: 't1', priority: 2, completed: false, tags: ['ui'] },
    { id: 't2', priority: 1, completed: false, tags: ['core'] },
    { id: 't3', priority: 3, completed: true, tags: ['docs'] }
  ]);

  // If implementation regresses from .toSorted() back to .sort(), Object.freeze throws TypeError!
  const activeSorted = sortAndFilterTasks(rawTasks);

  assert.deepEqual(
    activeSorted.map((t) => t.id),
    ['t2', 't1']
  );
});
```

---

## CSS, Wizz.js & Design System Geometry Regression

Avoid full-page HTML string snapshots (`expect(container.innerHTML).toMatchSnapshot()`), which break on every class rename while completely missing broken CSS layouts, clipped shadows, or missing focus rings. Instead, assert **computed style invariants** and **component-scoped visual screenshots**:

```javascript
import { test, expect } from '@playwright/test';

test('regression: Neo-Brutalist button obeys coupled translation + shadow law on press', async ({ page }) => {
  await page.goto('/components/buttons');
  const button = page.getByRole('button', { name: 'Deploy Now' });

  // 1. Verify accessibility tap target minimum (44x44px WCAG 2.2)
  const box = await button.boundingBox();
  expect(box.width).toBeGreaterThanOrEqual(44);
  expect(box.height).toBeGreaterThanOrEqual(44);

  // 2. Verify computed border & zero-blur shadow in idle state
  const idleStyles = await button.evaluate((el) => {
    const cs = getComputedStyle(el);
    return {
      borderTopWidth: cs.borderTopWidth,
      boxShadow: cs.boxShadow,
      outlineStyle: cs.outlineStyle
    };
  });
  expect(idleStyles.borderTopWidth).toBe('3px');
  expect(idleStyles.boxShadow).toContain('5px 5px 0px');

  // 3. Component-scoped visual regression with animations frozen
  await expect(button).toHaveScreenshot('nb-button-idle.png', {
    animations: 'disabled',
    maxDiffPixelRatio: 0.01
  });
});
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| Applying a bug fix first, then writing a passing test after | Write the failing test first (**Red**), then apply the fix (**Green**) | A test written after the fix has never been seen to fail and may be testing the wrong code path entirely. |
| Giant `toMatchSnapshot()` dumps of DOM trees or big JSON blobs | Targeted assertions on public contracts, invariants, and key fields | Developers habitually run `-u` (`--updateSnapshot`) when large snapshots fail, rubber-stamping actual regressions into git. |
| Reproducing race conditions with `setTimeout(10)` vs `setTimeout(50)` | Deterministic barriers with `Promise.withResolvers()` | Timer-based race tests pass locally and fail randomly in CI when CPU load shifts execution order. |
| Full-page `page.screenshot()` with live timestamps, random avatars, and CSS animations | Component-scoped `expect(locator).toHaveScreenshot({ animations: 'disabled' })` | Unscoped screenshots flake on every sub-pixel font shift, clock tick, or keyframe animation frame. |
| Deleting or `.skip()`-ing a failing regression test to unblock a merge | Diagnose whether the regression is real or the contract intentionally changed | Skipping failing tests hides breaking changes and erodes suite trustworthiness. |
