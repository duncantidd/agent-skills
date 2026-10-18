# Testing, Benchmarking, Regression & Review Skills

A comprehensive verification, performance benchmarking, regression prevention, and adversarial code review knowledge base engineered specifically for AI coding agents.

---

## Architecture: Why Testing Splits Into Two Layers

Testing standards naturally divide into **Language-Specific Runtime Standards** and **Cross-Language Engineering Workflows**:

```text
testing/
├── README.md                                          # Domain architecture & router index (this file)
│
├── javascript/                                        # Layer 1: Language-Specific Runtime Standards (JS/V8/Web)
│   ├── README.md                                      # JS verification principles & index
│   ├── js_testing_standards_skill_guide.md            # node:test, node:assert/strict, t.mock, Playwright locators
│   ├── js_benchmark_testing_skill_guide.md            # V8 JIT warmup, DCE sinks, p50/p95/p99, heap profiling
│   └── js_regression_testing_skill_guide.md           # Red-Before-Green, Promise.withResolvers races, CSS geometry
│
└── workflows/                                         # Layer 2: Cross-Language Review & Verification Loops
    ├── README.md                                      # Workflow principles & index
    ├── code_review_standards_skill_guide.md           # 4-Pass Adversarial Audit, severity taxonomy, Do Not checks
    └── testing_and_review_loops_skill_guide.md        # Closed-loop TDD, bugfix, benchmark, and self-review cycles
```

### Why Separate Language Standards from Workflows?
1. **Language & Runtime Mechanics Are Deeply Specific (`testing/<language>/`)**:  
   How you write a zero-dependency unit test, mock a timer, prevent JIT dead-code elimination in a microbenchmark, or assert DOM geometry depends entirely on the runtime:
   - In **JavaScript (`testing/javascript/`)**, agents must use `node:test`, `node:assert/strict`, `t.mock.timers`, `Promise.withResolvers()` race barriers, V8 TurboFan warmup loops, and Playwright semantic locators.
   - As future languages are added (e.g., `testing/python/`, `testing/rust/`, `testing/go/`), each gets its own runtime-native testing, benchmarking, and regression guides without muddying the JavaScript rules.
2. **Review & Closed-Loop Verification Protocols Are Universal (`testing/workflows/`)**:  
   How an agent conducts a **4-Pass Adversarial Code Review**, classifies findings (`BLOCKER`, `MAJOR`, `MINOR`, `NIT`), enforces **Red-Before-Green** bug reproduction, and runs a **Closed-Loop Test $\rightarrow$ Benchmark $\rightarrow$ Self-Review cycle** applies across every language and framework in the repository.

---

## Sub-Domain Index

### 1. ⚡ [JavaScript Native Testing, Benchmarking & Regression](./javascript/README.md)
| Guide | Focus Area | Key Capabilities |
| :--- | :--- | :--- |
| **[`js_testing_standards_skill_guide.md`](./javascript/js_testing_standards_skill_guide.md)** | Unit, Integration & DOM Testing | `node:test`, `node:assert/strict`, parameterized subtests, `t.mock.method()`, `t.mock.timers`, `t.signal` + `t.after()`, Playwright `getByRole()`. |
| **[`js_benchmark_testing_skill_guide.md`](./javascript/js_benchmark_testing_skill_guide.md)** | Statistical Benchmarking & Memory | `performance.now()`, V8 warmup, DCE blackhole sinks, `p50`/`p95`/`p99`/`RME%`, interleaved A/B batches, `process.memoryUsage().heapUsed`. |
| **[`js_regression_testing_skill_guide.md`](./javascript/js_regression_testing_skill_guide.md)** | Bug Reproduction & UI Invariants | Red-Before-Green protocol, `Promise.withResolvers()` concurrency barriers, `err.cause` assertions, `deepFreeze()` immutability checks, CSS/Wizz geometry regression. |

### 2. 🔄 [Engineering Workflows — Code Review & Verification Loops](./workflows/README.md)
| Guide | Focus Area | Key Capabilities |
| :--- | :--- | :--- |
| **[`code_review_standards_skill_guide.md`](./workflows/code_review_standards_skill_guide.md)** | Adversarial Code Review | `BLOCKER`/`MAJOR`/`MINOR`/`NIT` severity taxonomy, 4-Pass Adversarial Audit (Correctness, CSS/Wizz/Design Laws, Security, Test Mutation), structured review output. |
| **[`testing_and_review_loops_skill_guide.md`](./workflows/testing_and_review_loops_skill_guide.md)** | Closed-Loop Agent Execution | Feature TDD Loop, Red-Before-Green Bugfix Loop, A/B Benchmark Optimization Loop, Flake Elimination Loop, Pre-Completion Quality Gate. |

---

## Agent Retrieval & Usage Instructions

Whenever an AI agent is tasked with writing code, fixing a bug, optimizing performance, or reviewing a pull request:
1. **Load the Language Testing Guide**: For JavaScript, Wizz.js, or CSS tasks, consult [`testing/javascript/README.md`](./javascript/README.md).
2. **Load the Verification Workflow**: Consult [`testing/workflows/README.md`](./workflows/README.md) to execute the appropriate closed-loop test/review cycle before finalizing any response.
