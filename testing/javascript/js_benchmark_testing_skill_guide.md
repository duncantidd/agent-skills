# JavaScript Native — Benchmark Testing & Performance Verification

## Benchmark Testing Decision Matrix

| Goal | Tool | Why |
| :--- | :--- | :--- |
| High-resolution elapsed timing | `performance.now()` | Monotonic sub-millisecond clock unaffected by system clock drift or NTP adjustments (`Date.now()` is non-monotonic and 1ms-coarse). |
| Preventing JIT dead-code elimination | Module-scoped accumulator / blackhole sink | V8's TurboFan JIT optimizes away pure loop computations if their return values are never consumed outside the loop. |
| Measuring statistical distribution | Warmup phase + Sampled p50 / p95 / p99 / MAD | Single-run averages hide GC pauses and JIT compilation spikes; median (p50) and tail latencies (p99) reveal true runtime behavior. |
| Tracking memory & GC allocation pressure | `process.memoryUsage().heapUsed` (with `--expose-gc`) | Reveals hidden object allocations and garbage-collection thrashing that micro-timings alone miss on short runs. |
| Automated A/B baseline comparison | Interleaved (paired) benchmark trials | Running Candidate A and Candidate B in alternating batches cancels out background OS/CPU thermal throttling drift. |
| Measuring structured execution phases | `performance.mark()` + `performance.measure()` | Standard Web/Node User Timing API observable via `PerformanceObserver` without polluting business logic. |
| Browser rendering & UI responsiveness | `PerformanceObserver` (`long-animation-frame` / `layout-shift`) | Captures main-thread blocking frames (>50ms) and forced synchronous layouts in real browser engines. |

---

## V8 / JIT-Aware Microbenchmark Harness

AI agents frequently write naive `const t0 = Date.now(); for (...) fn();` loops that produce meaningless numbers because:
1. **Cold JIT distortion**: The first few thousand iterations measure bytecode interpretation (Ignition) and compilation overhead rather than steady-state optimized machine code (TurboFan).
2. **Dead-Code Elimination (DCE)**: If the return value of `fn()` is unused, V8 hoists invariants or eliminates the computation entirely.
3. **Outlier skew**: Arithmetic mean is skewed by single OS context switches; always report **p50 (median)**, **p95**, **p99**, and **Relative Margin of Error (RME)**.

```javascript
import { performance } from 'node:perf_hooks';

// Exported sink prevents V8 TurboFan from eliminating pure return values
export let __benchmarkSink = 0;

/**
 * Runs a statistically sound, JIT-warmed benchmark with dead-code elimination protection.
 * @param {string} name
 * @param {() => unknown} fn - Operation under test (must return a value)
 * @param {{ warmupMs?: number, samples?: number, batchSize?: number }} [options]
 */
export function runBenchmark(name, fn, { warmupMs = 300, samples = 100, batchSize = 1000 } = {}) {
  // 1. Warmup Phase: Allow V8 Ignition -> Sparkplug -> TurboFan tier-up
  const warmupDeadline = performance.now() + warmupMs;
  while (performance.now() < warmupDeadline) {
    for (let i = 0; i < batchSize; i++) {
      __benchmarkSink ^= Number(Boolean(fn()));
    }
  }

  // 2. Sampled Measurement Phase: Record per-operation nanoseconds across batches
  const timingsNs = new Float64Array(samples);
  for (let s = 0; s < samples; s++) {
    const startMs = performance.now();
    for (let i = 0; i < batchSize; i++) {
      const res = fn();
      if (res !== undefined) __benchmarkSink ^= 1;
    }
    const elapsedNs = (performance.now() - startMs) * 1e6;
    timingsNs[s] = elapsedNs / batchSize;
  }

  // 3. Immutable sort for percentile calculation
  const sorted = Array.from(timingsNs).toSorted((a, b) => a - b);
  const p50 = sorted[Math.floor(samples * 0.50)];
  const p95 = sorted[Math.floor(samples * 0.95)];
  const p99 = sorted[Math.floor(samples * 0.99)];

  // Mean & Standard Error for Relative Margin of Error (RME %)
  const mean = sorted.reduce((acc, v) => acc + v, 0) / samples;
  const variance = sorted.reduce((acc, v) => acc + (v - mean) ** 2, 0) / (samples - 1);
  const stdErr = Math.sqrt(variance / samples);
  const rmePercent = (stdErr * 1.96 / mean) * 100;

  return {
    name,
    opsPerSec: Math.round(1e9 / p50),
    p50Ns: Number(p50.toFixed(1)),
    p95Ns: Number(p95.toFixed(1)),
    p99Ns: Number(p99.toFixed(1)),
    rmePercent: Number(rmePercent.toFixed(2))
  };
}
```

---

## Interleaved A/B Baseline Comparison

Never run all iterations of Implementation A followed by all iterations of Implementation B—CPU thermal throttling makes the second function appear artificially slower. **Interleave batches** and vary input fixtures from a pre-generated pool to prevent monomorphic inline-cache cheating:

```javascript
import assert from 'node:assert/strict';
import { test } from 'node:test';
import { performance } from 'node:perf_hooks';

export function compareImplementations(baselineFn, candidateFn, inputs, { rounds = 60, batchSize = 500 } = {}) {
  const baselineSamples = [];
  const candidateSamples = [];
  const inputLen = inputs.length;

  // Alternate order each round (A->B on even rounds, B->A on odd rounds)
  for (let r = 0; r < rounds; r++) {
    const measureBatch = (fn) => {
      const t0 = performance.now();
      for (let i = 0; i < batchSize; i++) {
        fn(inputs[(r + i) % inputLen]);
      }
      return ((performance.now() - t0) * 1e6) / batchSize;
    };

    if (r % 2 === 0) {
      baselineSamples.push(measureBatch(baselineFn));
      candidateSamples.push(measureBatch(candidateFn));
    } else {
      candidateSamples.push(measureBatch(candidateFn));
      baselineSamples.push(measureBatch(baselineFn));
    }
  }

  const median = (arr) => arr.toSorted((a, b) => a - b)[Math.floor(arr.length / 2)];
  const baseP50 = median(baselineSamples);
  const candP50 = median(candidateSamples);

  return {
    baselineP50Ns: baseP50,
    candidateP50Ns: candP50,
    speedupRatio: Number((baseP50 / candP50).toFixed(2)),
    deltaPercent: Number((((candP50 - baseP50) / baseP50) * 100).toFixed(2))
  };
}

// Example performance regression gate in node:test
test('Set lookup outperforms Array.includes for >100 item membership checks', () => {
  const allowedIds = Array.from({ length: 500 }, (_, i) => `item_${i}`);
  const allowedSet = new Set(allowedIds);
  const queries = ['item_10', 'item_250', 'item_499', 'missing_id'];

  const comparison = compareImplementations(
    (q) => allowedIds.includes(q),
    (q) => allowedSet.has(q),
    queries
  );

  // Verify candidate is significantly faster (speedupRatio > 2x)
  assert.equal(comparison.speedupRatio > 2, true, `Expected >2x speedup, got ${comparison.speedupRatio}x`);
});
```

---

## Memory Allocation & GC Pressure Profiling

A function that runs 5% faster in a tiny loop by allocating temporary arrays on every call can degrade production throughput by triggering stop-the-world Garbage Collection pauses. Run Node with `--expose-gc` during memory benchmarks to measure net heap allocation per operation:

```javascript
/**
 * Measures average bytes allocated on the V8 heap per invocation.
 * Run with: node --expose-gc bench.js
 */
export function measureHeapAllocation(fn, iterations = 10_000) {
  if (typeof globalThis.gc === 'function') {
    globalThis.gc();
  }

  const beforeHeap = process.memoryUsage().heapUsed;
  for (let i = 0; i < iterations; i++) {
    fn(i);
  }
  const afterHeap = process.memoryUsage().heapUsed;

  const totalBytesDelta = Math.max(0, afterHeap - beforeHeap);
  return {
    totalBytesDelta,
    bytesPerOp: Math.round(totalBytesDelta / iterations)
  };
}
```

---

## Structured Macro-Benchmarking (`PerformanceObserver` & User Timing)

For asynchronous pipelines, database queries, or SSR rendering passes, instrument code using the standard Web/Node `performance.mark()` and `performance.measure()` APIs:

```javascript
import { performance, PerformanceObserver } from 'node:perf_hooks';

export async function profileAsyncPipeline(stepName, asyncWork) {
  const startMark = `${stepName}:start`;
  const endMark = `${stepName}:end`;

  const { promise, resolve } = Promise.withResolvers();
  const observer = new PerformanceObserver((list) => {
    const entry = list.getEntriesByName(stepName).at(-1);
    if (entry) {
      observer.disconnect();
      resolve({
        name: entry.name,
        durationMs: Number(entry.duration.toFixed(3)),
        detail: entry.detail
      });
    }
  });

  observer.observe({ entryTypes: ['measure'] });
  performance.mark(startMark);

  try {
    const result = await asyncWork();
    return { result, metrics: await promise };
  } finally {
    performance.mark(endMark);
    performance.measure(stepName, { start: startMark, end: endMark });
    performance.clearMarks(startMark);
    performance.clearMarks(endMark);
  }
}
```

---

## Browser UI & Frame-Budget Benchmarking (`16.6ms` / `60fps` Gate)

When benchmarking DOM updates, Wizz.js reactive list reconciliations, or CSS animations, verify that main-thread work stays within the `16.67ms` frame budget and triggers zero Long Animation Frames (`>50ms`):

```javascript
import { test, expect } from '@playwright/test';

test('rendering 1,000 reactive rows completes under 50ms without forced layout thrashing', async ({ page }) => {
  await page.goto('/data-grid');

  const timing = await page.evaluate(async () => {
    performance.mark('render-start');
    window.dispatchEvent(new CustomEvent('benchmark:populate-rows', { detail: { count: 1000 } }));

    // Wait for next animation frame after paint
    await new Promise((resolve) => requestAnimationFrame(() => requestAnimationFrame(resolve)));
    performance.mark('render-end');

    const measure = performance.measure('grid-render-1k', 'render-start', 'render-end');
    return { durationMs: measure.duration };
  });

  expect(timing.durationMs).toBeLessThan(50);
});
```

---

## Do Not

| BAD | GOOD | Why |
| :--- | :--- | :--- |
| `const t0 = Date.now(); fn(); Date.now() - t0` | `performance.now()` across warmed batches | `Date.now()` has only millisecond precision and jumps backward/forward during NTP clock syncs. |
| Benchmarking a cold loop without warmup | Warm up for `200ms–500ms` before recording samples | Cold iterations measure V8 parser and Ignition bytecode overhead instead of optimized machine code. |
| Calling a pure function in a loop and ignoring its return value | Feed return values into an exported accumulator sink | V8's TurboFan dead-code eliminator deletes unused pure computations, reporting fake `0.001ns` times. |
| Benchmarking with a single hardcoded literal input (`fn("abc")`) | Rotate inputs from a pre-generated array of realistic fixtures | A single constant input allows V8 to constant-fold the entire branch, misrepresenting production polymorphism. |
| Reporting only a single arithmetic mean (`avgMs`) | Report `p50` (median), `p95`, `p99`, and `RME %` | A single GC pause skews the arithmetic mean; p50 and p99 distinguish baseline speed from tail jitter. |
| Running all of Test A then all of Test B | Interleave batches (`A, B, B, A...`) in the same process | CPU boost clocks decay after seconds of load, systematically penalizing whichever implementation runs second. |
