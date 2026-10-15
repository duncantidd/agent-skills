# JavaScript Native — Modern Standard Library & Zero-Dependency Utilities

## Standard Library Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| Deep cloning objects | `structuredClone()` | Native, handles cyclic references, Maps, Sets, Dates, and ArrayBuffers without lodash or JSON serialization quirks. |
| Grouping collection items | `Object.groupBy()` / `Map.groupBy()` | Native grouping by criterion without `lodash.groupBy` or verbose `Array.prototype.reduce`. |
| Sorting without array mutation | `array.toSorted()` | Returns a newly sorted copy without mutating the source array in place. |
| Reversing without mutation | `array.toReversed()` | Returns a reversed copy while keeping original array intact. |
| Splicing without mutation | `array.toSpliced()` | Immutably removes/inserts elements and returns the resulting copy. |
| Single index replacement | `array.with(index, value)` | Immutably returns a new array with one item updated. |
| Reading from array end | `array.at(-1)` | Negative index lookup without `array[array.length - 1]`. |
| Searching from array end | `array.findLast()` / `findLastIndex()` | Scans backwards directly without reversing the array first. |
| Generating UUID v4 | `crypto.randomUUID()` | Cryptographically secure UUID generation native in all modern browsers and Node 14.17+. |
| External promise resolution | `Promise.withResolvers()` | Deconstructs `promise`, `resolve`, and `reject` without awkward closure extraction. |

---

## Deep Object Cloning: `structuredClone()`

Never use `JSON.parse(JSON.stringify(obj))`—it drops `undefined`, destroys `Date` objects into strings, drops `Map` and `Set`, and throws fatal errors on circular references.

```javascript
const original = {
  id: 101,
  created: new Date(),
  tags: new Set(['admin', 'billing']),
  metadata: new Map([['region', 'us-east']])
};

// Complete deep clone preserving built-in types
const clone = structuredClone(original);

clone.tags.add('owner');
console.log(original.tags.has('owner')); // false (isolated copy)
```

---

## Native Collection Grouping (`Object.groupBy` & `Map.groupBy`)

Group arrays of items into categorized objects or maps without `lodash.groupBy`:

```javascript
const inventory = [
  { name: 'Laptop', category: 'electronics', stock: 5 },
  { name: 'Shirt', category: 'apparel', stock: 12 },
  { name: 'Mouse', category: 'electronics', stock: 0 }
];

// 1. Group into a plain object keyed by string
const byCategory = Object.groupBy(inventory, item => item.category);
/* Output:
{
  electronics: [{ name: 'Laptop', ... }, { name: 'Mouse', ... }],
  apparel: [{ name: 'Shirt', ... }]
}
*/

// 2. Group into a Map (supports complex keys like objects or booleans)
const byStockStatus = Map.groupBy(inventory, item => item.stock > 0);
console.log(byStockStatus.get(true));  // In-stock items
console.log(byStockStatus.get(false)); // Out-of-stock items
```

---

## Immutability: Non-Mutating Array Methods

Traditional array methods (`sort`, `reverse`, `splice`) mutate arrays in-place, causing unpredictable bugs in reactive UI frameworks (React, Vue) and functional data pipelines.

```javascript
const scores = [85, 42, 99, 71];

// 1. Immutable sort (replaces [...scores].sort())
const sortedAsc = scores.toSorted((a, b) => a - b);

// 2. Immutable reverse (replaces [...scores].reverse())
const reversed = scores.toReversed();

// 3. Immutable splice (replaces manual slice/spread surgery)
// toSpliced(start, deleteCount, ...itemsToAdd)
const withoutSecond = scores.toSpliced(1, 1); // removes index 1

// 4. Immutable single-index update
const updated = scores.with(0, 100); // replaces index 0 with 100

// Original array remains untouched:
console.log(scores); // [85, 42, 99, 71]
```

---

## Array Navigation: `.at()`, `.findLast()`, and `.findLastIndex()`

Stop computing `array.length - 1` or reversing arrays just to inspect the tail:

```javascript
const versions = ['v1.0.0', 'v1.1.0', 'v2.0.0-rc', 'v2.0.0'];

// Read from end using negative index
const latest = versions.at(-1);      // 'v2.0.0'
const secondLatest = versions.at(-2); // 'v2.0.0-rc'

// Search from tail backwards
const logs = [
  { id: 1, status: 'fail' },
  { id: 2, status: 'pass' },
  { id: 3, status: 'fail' },
  { id: 4, status: 'pass' }
];

const lastFailed = logs.findLast(log => log.status === 'fail');
// { id: 3, status: 'fail' }

const lastFailedIndex = logs.findLastIndex(log => log.status === 'fail');
// 2
```

---

## Cryptographic UUIDs: `crypto.randomUUID()`

Do not install the `uuid` package. `crypto.randomUUID()` is built into all modern browsers, Node.js (14.17+), Deno, and Bun:

```javascript
// Native RFC 4122 Version 4 UUID
const transactionId = crypto.randomUUID();
// e.g. "36b8f84d-df4e-4d49-b662-bcde71a8764f"
```

---

## Deferred Promises: `Promise.withResolvers()`

Eliminates the awkward pattern of declaring `let resolve, reject` outside the `new Promise` constructor:

```javascript
// Modern standard (ES2024)
const { promise, resolve, reject } = Promise.withResolvers();

// Pass resolve/reject to external callbacks, workers, or event handlers
document.getElementById('confirm-btn').addEventListener('click', () => {
  resolve({ confirmed: true });
}, { once: true });

document.getElementById('cancel-btn').addEventListener('click', () => {
  reject(new Error('User cancelled'));
}, { once: true });

// Await elsewhere cleanly
const result = await promise;
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| `JSON.parse(JSON.stringify(x))` | `structuredClone(x)` | Destroys Dates, Sets, Maps, functions, and crashes on circular references. |
| `import _ from 'lodash'` for basic utilities | Use native ES2020-ES2024 methods | Bloats bundle size by tens of kilobytes for methods now built into the runtime. |
| `array.sort()` on reactive state | `array.toSorted()` | In-place mutation mutates props/state directly, preventing re-renders in UI frameworks. |
| `array[array.length - 1]` | `array.at(-1)` | Verbose, error-prone, and clunky on chained function calls. |
| `import { v4 } from 'uuid'` | `crypto.randomUUID()` | Unnecessary third-party dependency; native version is faster and zero-bundle-size. |
| `let res, rej; new Promise((r1, r2) => { res = r1; rej = r2; })` | `Promise.withResolvers()` | Eliminates closure boilerplate and variable scoping hazards. |
