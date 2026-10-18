# JavaScript Native — Testing Standards & Zero-Dependency Verification

## Testing Standards Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Running unit & integration tests | `node:test` (`node --test`) | Built into Node.js (18+); zero npm dependencies, native ESM support, parallel execution out of the box. |
| Writing test assertions | `node:assert/strict` | Enforces strict equality (`Object.is` / deep strict comparison) without Chai/Expect bloat or loose coercion bugs. |
| Mocking functions & methods | `t.mock.fn()` / `t.mock.method()` | Scoped to the test lifecycle `t`; automatically restores originals after the test completes, preventing cross-test pollution. |
| Controlling time (`setTimeout`, `Date`) | `t.mock.timers.enable()` | Deterministically advances virtual clocks in microseconds instead of adding slow, flaky real `setTimeout` sleeps. |
| Enforcing test timeouts & cleanup | `{ signal: t.signal }` + `t.after()` | Ties async operations (`fetch`, streams, event listeners) directly to the test's `AbortSignal` for instant teardown. |
| Parameterized / table-driven cases | `for (const tc of cases) await t.test(...)` | Standard JavaScript loops with subtests provide clear failure isolation and native stack traces without custom DSLs. |
| Browser DOM / UI & E2E verification | Playwright (`@playwright/test`) + semantic locators | Real headless browser engines with role-based accessibility locators (`getByRole`, `getByLabel`) instead of brittle JSDOM mocks or CSS class selectors. |

---

## Zero-Dependency Test Architecture (`node:test` & `node:assert/strict`)

Do not install `jest`, `mocha`, `chai`, `sinon`, or `ts-jest` for standard JavaScript modules. Node's built-in test runner (`node:test`) and strict assertion module (`node:assert/strict`) execute ES modules natively with zero configuration.

```javascript
import { describe, test } from 'node:test';
import assert from 'node:assert/strict';
import { calculateInvoiceTotal } from '../src/billing.js';

describe('calculateInvoiceTotal()', () => {
  test('computes subtotal, tax, and discount without floating-point drift', () => {
    const lineItems = [
      { id: 'item-1', priceCents: 1999, quantity: 2 },
      { id: 'item-2', priceCents: 500, quantity: 1 }
    ];

    const result = calculateInvoiceTotal(lineItems, {
      discountCents: 300,
      taxRateBasisPoints: 825 // 8.25%
    });

    assert.deepEqual(result, {
      subtotalCents: 4498,
      discountCents: 300,
      taxableCents: 4198,
      taxCents: 346,
      totalCents: 4544
    });
  });

  test('throws RangeError with cause when quantity is negative', () => {
    const invalidItems = [{ id: 'item-1', priceCents: 1000, quantity: -1 }];

    assert.throws(
      () => calculateInvoiceTotal(invalidItems),
      (err) => {
        assert.equal(err instanceof RangeError, true);
        assert.match(err.message, /quantity must be non-negative/i);
        return true;
      }
    );
  });
});
```

---

## Table-Driven Parameterized Subtests

Test boundary conditions, malformed inputs, and equivalence classes systematically using table-driven subtests (`await t.test()`). Each row runs as an isolated subtest with a descriptive name:

```javascript
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { parseSlug } from '../src/slug.js';

test('parseSlug() boundary & Unicode normalization matrix', async (t) => {
  const cases = [
    { name: 'ASCII title', input: 'Hello World!', expected: 'hello-world' },
    { name: 'Accented characters', input: 'Café Crème & Thé', expected: 'cafe-creme-the' },
    { name: 'Collapses multiple dashes', input: 'foo---bar   baz', expected: 'foo-bar-baz' },
    { name: 'Trims leading/trailing separators', input: ' ---clean-slug--- ', expected: 'clean-slug' },
    { name: 'Empty whitespace returns empty string', input: '    ', expected: '' }
  ];

  for (const { name, input, expected } of cases) {
    await t.test(name, () => {
      assert.equal(parseSlug(input), expected);
    });
  }
});
```

---

## Scoped Mocking & Fake Timers (`t.mock`)

Global mocks leak across tests when an assertion throws before `.mockRestore()` runs. Always bind mocks and fake timers to the test context `t` so Node automatically restores them on test exit.

### 1. Scoped Method Spies & Dependency Injection
Prefer passing explicit dependencies (or interfaces) over monkey-patching module internals. When spying on an object method, use `t.mock.method()`:

```javascript
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { syncUserProfile } from '../src/user-sync.js';

test('syncUserProfile() retries on 503 and preserves error cause on exhaustion', async (t) => {
  const apiClient = {
    async fetchProfile(id) {
      throw new Error('Real network call should not execute');
    }
  };

  let attempts = 0;
  const spy = t.mock.method(apiClient, 'fetchProfile', async (id) => {
    attempts += 1;
    if (attempts < 3) {
      const err = new Error('Service Unavailable');
      err.status = 503;
      throw err;
    }
    return { id, name: 'Ada Lovelace' };
  });

  const profile = await syncUserProfile('usr_42', { client: apiClient, retryDelayMs: 0 });

  assert.equal(spy.mock.callCount(), 3);
  assert.deepEqual(spy.mock.calls[0].arguments, ['usr_42']);
  assert.deepEqual(profile, { id: 'usr_42', name: 'Ada Lovelace' });
});
```

### 2. Deterministic Virtual Clocks (`t.mock.timers`)
Never use `await new Promise(r => setTimeout(r, 500))` to test debouncing, throttling, TTL caches, or retry backoffs. Enable scoped fake timers and tick the virtual clock deterministically:

```javascript
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { createDebounce } from '../src/debounce.js';

test('createDebounce() coalesces rapid calls and fires only after quiet window', (t) => {
  t.mock.timers.enable({ apis: ['setTimeout', 'Date'] });

  const events = [];
  const debouncedSave = createDebounce((val) => events.push(val), 300);

  debouncedSave('draft-1');
  t.mock.timers.tick(150); // 150ms elapsed — should not fire yet

  debouncedSave('draft-2');
  t.mock.timers.tick(299); // 299ms since second call — still quiet
  assert.equal(events.length, 0);

  t.mock.timers.tick(1);   // 300ms reached — fires with latest argument
  assert.deepEqual(events, ['draft-2']);
});
```

---

## Async Cancellation, Cleanup & Resource Isolation

Every test that opens a server, spawns a worker, attaches DOM listeners, or creates a temporary directory must register a deterministic teardown via `t.after()` and propagate `t.signal`:

```javascript
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { createServer } from 'node:http';
import { once } from 'node:events';

test('SSE stream aborts cleanly when client disconnects', { timeout: 2000 }, async (t) => {
  const server = createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/event-stream' });
    res.write('data: ping\n\n');
  });

  server.listen(0);
  await once(server, 'listening');

  // Guaranteed teardown even if assertions fail mid-test
  t.after(() => server.close());

  const { port } = server.address();
  const controller = new AbortController();

  // Combine test timeout signal with local test controller
  const signal = AbortSignal.any([t.signal, controller.signal]);

  const response = await fetch(`http://127.0.0.1:${port}/events`, { signal });
  assert.equal(response.status, 200);

  // Abort client stream and verify clean rejection
  controller.abort(new Error('Client closed tab'));
  await assert.rejects(
    () => response.text(),
    { name: 'Error', message: 'Client closed tab' }
  );
});
```

---

## DOM & UI Component Testing: Semantic Locators Over Implementation Details

When testing browser UI components (Vanilla DOM, Web Components, or Wizz.js), test **observable user-facing behavior and accessibility contracts** rather than internal class names or private state variables:

```javascript
import { test, expect } from '@playwright/test';

test('checkout modal validates invalid email and submits accessible form data', async ({ page }) => {
  await page.goto('/checkout');

  // 1. Query by accessible role and label — never by fragile CSS class chains
  await page.getByRole('button', { name: 'Proceed to Payment' }).click();

  const dialog = page.getByRole('dialog', { name: 'Payment Details' });
  await expect(dialog).toBeVisible();

  const emailInput = dialog.getByLabel('Receipt Email');
  await emailInput.fill('not-an-email');
  await emailInput.blur();

  // 2. Verify semantic validity state and user-visible error message
  await expect(emailInput).toHaveAttribute('aria-invalid', 'true');
  await expect(dialog.getByRole('alert')).toContainText('Enter a valid email address');

  // 3. Fix input and verify submission
  await emailInput.fill('ada@analytical-engine.org');
  await dialog.getByRole('button', { name: 'Confirm Order' }).click();
  await expect(dialog).toBeHidden();
});
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| Installing `jest` + `babel` for pure ESM JS | `node --test` (`node:test` + `node:assert/strict`) | Jest requires experimental flags or heavy transforms for native ESM and adds hundreds of transitive packages. |
| `import assert from 'node:assert'` (`assert.equal(1, '1')`) | `import assert from 'node:assert/strict'` | Legacy `assert.equal` uses loose `==` coercion (`1 == '1'` passes silently); `strict` enforces `Object.is` / deep strict equality. |
| `await new Promise(r => setTimeout(r, 1000))` in unit tests | `t.mock.timers.enable()` + `t.mock.timers.tick(1000)` | Real sleeps make test suites slow and cause non-deterministic CI failures under CPU contention. |
| Manual `const orig = obj.fn; obj.fn = ...` | `t.mock.method(obj, 'fn', impl)` | If an assertion throws before restoration, manual monkey-patches corrupt every subsequent test in the file. |
| Querying DOM by `.btn-primary.css-1x9a2` | `getByRole('button', { name: '...' })` | Testing CSS utility classes couples tests to styling refactors and misses broken accessibility labels. |
| Asserting on private fields or internal call counts of pure helpers | Asserting on public return values, state transitions, and boundary side effects | Over-mocking internal functions makes tests break on every harmless refactor while missing real integration bugs. |
| `try { await fn(); assert.fail(); } catch (e) {}` | `await assert.rejects(() => fn(), { name: 'TypeError' })` | Swallowing errors in `catch` without verifying error type/message lets broken test code pass by accident. |
