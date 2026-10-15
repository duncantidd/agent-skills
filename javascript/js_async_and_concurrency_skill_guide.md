# JavaScript Native — Async Patterns & Concurrency Control

## Async Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Sequential async execution | `for...of` loop with `await` | Guarantees iteration $N+1$ waits for iteration $N$ to complete. |
| Concurrent execution (fail-fast) | `Promise.all()` | Executes tasks in parallel; rejects immediately upon the first failure. |
| Concurrent execution (resilient) | `Promise.allSettled()` | Runs all tasks to completion and reports status (`fulfilled`/`rejected`) for each without aborting. |
| First successful response | `Promise.any()` | Resolves as soon as any promise succeeds; ignores rejections unless *all* fail. |
| First completed response (race) | `Promise.race()` | Resolves or rejects as soon as the first promise settles. |
| Automatic request/task timeout | `AbortSignal.timeout(ms)` | Built-in timeout signal; replaces manual `setTimeout` rejection races. |
| Manual or user-triggered cancellation | `AbortController` | Standard cross-platform cancellation interface supported by `fetch`, streams, and events. |
| Combining multiple cancel triggers | `AbortSignal.any([sig1, sig2])` | Triggers cancellation if *either* signal fires (e.g. timeout OR user cancel). |

---

## The `forEach` Async Trap

`Array.prototype.forEach` does **not** await async callbacks. It fires all iterations concurrently without waiting for any promise to settle:

```javascript
// ❌ WRONG: Executes concurrently, does not pause execution, and errors cannot be caught!
items.forEach(async (item) => {
  await saveToDb(item);
});
console.log('Done!'); // Prints immediately BEFORE database writes finish!

// ✅ GOOD (Sequential): When order matters or to avoid rate limits
for (const item of items) {
  await saveToDb(item);
}
console.log('All saved sequentially!');

// ✅ GOOD (Parallel): When tasks are independent and can run simultaneously
await Promise.all(items.map(item => saveToDb(item)));
console.log('All saved in parallel!');
```

---

## Promise Combinators: Fail-Fast vs Resilient Batching

### 1. `Promise.all` vs `Promise.allSettled`
If one request in 100 fails:
- `Promise.all` throws immediately, discarding the results of the 99 successful operations.
- `Promise.allSettled` waits for all 100 to finish, returning full diagnostics.

```javascript
const userIds = [1, 2, 99999]; // 99999 will 404

const results = await Promise.allSettled(
  userIds.map(id => fetchUser(id))
);

for (const result of results) {
  if (result.status === 'fulfilled') {
    console.log('User loaded:', result.value);
  } else {
    console.error('Failed to load user:', result.reason);
  }
}
```

---

## Cancellation & Timeout with `AbortController`

Never leave asynchronous operations without a cancellation or timeout path.

### 1. Standard Request Timeout
```javascript
// Native 5-second timeout on fetch
try {
  const response = await fetch('/api/data', {
    signal: AbortSignal.timeout(5000) // Automatically aborts after 5,000ms
  });
  const data = await response.json();
} catch (err) {
  if (err.name === 'TimeoutError') {
    console.error('Request timed out after 5s');
  } else {
    console.error('Network error:', err);
  }
}
```

### 2. Manual Cancellation
```javascript
const controller = new AbortController();

// Pass signal to any async task or fetch
const downloadPromise = fetch('/api/large-file', { signal: controller.signal });

// Cancel on user click
cancelButton.addEventListener('click', () => {
  controller.abort('User clicked cancel');
});
```

### 3. Combining Cancellation Signals (`AbortSignal.any`)
Cancel when *either* the user hits cancel OR a 10s deadline expires:

```javascript
const userCancelController = new AbortController();
const timeoutSignal = AbortSignal.timeout(10000);

// Aborts if user aborts OR if 10 seconds pass
const combinedSignal = AbortSignal.any([
  userCancelController.signal,
  timeoutSignal
]);

await fetch('/api/heavy-job', { signal: combinedSignal });
```

---

## Streaming Data with Async Iterators

Read large responses or streams chunk-by-chunk without loading the entire payload into RAM:

```javascript
async function consumeStream(url) {
  const response = await fetch(url);
  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  try {
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      const textChunk = decoder.decode(value, { stream: true });
      processChunk(textChunk);
    }
  } finally {
    reader.releaseLock();
  }
}
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| `items.forEach(async (x) => ...)` | `for (const x of items)` or `Promise.all(items.map(...))` | `forEach` ignores promises, runs out of order, and swallows errors. |
| `Promise.all` for batch background sync | `Promise.allSettled` | A single network error or invalid item should not abort an entire batch sync. |
| Custom timeout promises using `new Promise(setTimeout)` | `AbortSignal.timeout(ms)` | Native signal cleans up internal timer automatically and integrates with `fetch`. |
| Uncancelable long-running async tasks | Pass `AbortSignal` | Uncancelable operations cause memory leaks, race conditions, and wasted bandwidth. |
| Catching errors and returning `null` silently | Re-throw with `cause` or return structured result | Silent failure turns asynchronous exceptions into mysterious downstream `TypeError: Cannot read property`. |
| Mixing callbacks and promises | Stick exclusively to `async/await` | Callback-promise hybrids cause double-execution and broken stack traces. |
