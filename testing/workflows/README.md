# Engineering Workflows — Code Review & Verification Loops

A cross-language verification workflow knowledge base engineered specifically for AI coding agents. While language-specific test runners and JIT benchmark harnesses live under [`../javascript/`](../javascript/README.md) (and future language subdirectories), this directory defines the **universal engineering protocols** for adversarial code review and closed-loop test/review execution.

---

## Mission & Purpose

AI coding agents frequently fail at verification not because they lack syntax knowledge, but because their execution workflow is open-loop:
- Writing implementation code and immediately reporting completion without running tests or checking edge cases.
- Performing polite, superficial code reviews ("LGTM, clean structure!") that miss race conditions, XSS sinks, memory leaks, and violations of repository `Do Not` tables.
- Patching bugs before writing a failing reproduction test (**Red-Before-Green**), leaving no proof that the test actually catches the bug.
- Claiming performance wins without running warmed, interleaved A/B benchmarks.

This directory provides deterministic, closed-loop workflows that every agent must execute when writing, fixing, optimizing, or reviewing code.

---

## Core Workflow Principles

1. **Adversarial Multi-Pass Code Review**  
   Assume every diff contains at least one boundary bug, concurrency hazard, or domain anti-pattern. Audit systematically across 4 passes: (1) Correctness & Concurrency, (2) CSS/Wizz/Design System Laws, (3) Security & Trust Boundaries, and (4) Test Mutation Sufficiency.

2. **Explicit Severity Triage (`BLOCKER` / `MAJOR` / `MINOR` / `NIT`)**  
   Tag every review finding with its severity level, file/line location, concrete counterexample trigger, and a drop-in code remediation.

3. **Closed-Loop Execution (Never Fire-and-Forget)**  
   Every feature, bugfix, or optimization must pass through an iterative verification loop: write or update the test, run the suite, audit the diff against domain `Do Not` tables, and resolve all `[BLOCKER]` and `[MAJOR]` findings before finishing.

4. **Strict Red-Before-Green Bug Remediation**  
   Never touch production code for a bug fix until an automated regression test has been written and observed failing with the exact symptom.

---

## Directory Index

| Guide | Description | Key Workflows & Tools |
| :--- | :--- | :--- |
| **[`code_review_standards_skill_guide.md`](./code_review_standards_skill_guide.md)** | Adversarial code review methodology & severity taxonomy | `BLOCKER`/`MAJOR`/`MINOR`/`NIT` triage, 4-Pass Adversarial Audit, `agent-skills` `Do Not` table enforcement, Mutation Thought-Experiment for tests, structured review output schema. |
| **[`testing_and_review_loops_skill_guide.md`](./testing_and_review_loops_skill_guide.md)** | Closed-loop autonomous test, benchmark, and self-review cycles | Loop 1 (Feature TDD + Self-Review), Loop 2 (Red-Before-Green Bugfix), Loop 3 (Interleaved A/B Benchmark Optimization), Loop 4 (Flake Elimination), Loop 5 (Pre-Completion Quality Gate). |

---

## Agent Usage Instructions

1. **When Reviewing Code or PRs**: Load [`code_review_standards_skill_guide.md`](./code_review_standards_skill_guide.md) and execute all 4 adversarial passes before emitting a structured review summary.
2. **When Implementing Features, Fixes, or Optimizations**: Load [`testing_and_review_loops_skill_guide.md`](./testing_and_review_loops_skill_guide.md) alongside the target language's testing guides (e.g., [`../javascript/README.md`](../javascript/README.md)) to drive your implementation through a closed verification loop.
