---
name: sci-review
version: 1.0.0
description: |
  Algorithm, scientific-computing, AI and data science plan review. Lock in the execution plan —
  correctness, convergence, numerical stability, complexity, edge cases, benchmarks.
  Walks through issues interactively with opinionated recommendations. Use when asked
  to "review the algorithm", "review the solver design", "scientific review", or
  "lock in the plan" for computational/algorithmic/AI/data science projects.
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - AskUserQuestion
  - Bash
  - WebSearch
---

# Scientific / Algorithmic Plan Review

**Announce at start**: "I'm using the sci-review skill."

Review this plan thoroughly before making any code changes. For every issue or recommendation, explain the concrete tradeoffs, give an opinionated recommendation, and ask for input before assuming a direction.

## Bootstrap: Read Rules

Before starting the review, read these rules to inform your evaluation:

| Rule file | Why |
|---|---|
| `.claude/skills/control-tower/rules/TDD.md` | Test design framework for algorithmic components |
| `.claude/skills/control-tower/rules/KEEP_IN_MIND.md` | Interface design and testability principles |

**Procedure:** Read all listed rule files at the start. If any file is missing, note it and continue.

## My engineering preferences (use these to guide your recommendations):

- Correctness is non-negotiable — a fast wrong answer is worthless.
- Well-tested code is non-negotiable; I'd rather have too many tests than too few.
- I want code that's "engineered enough" — not under-engineered (fragile, hacky) and not over-engineered (premature abstraction, unnecessary complexity).
- I err on the side of handling more edge cases, not fewer; thoughtfulness > speed.
- Bias toward explicit over clever. Readable math > terse math.
- Minimal diff: achieve the goal with the fewest new abstractions and files touched.
- DRY is important — flag repetition aggressively.

## Cognitive Patterns — How Strong Algorithm Designers Think

These are not checklist items. They are the instincts that experienced computational engineers develop. Apply them throughout your review.

1. **Correctness first, optimize later** — A correct brute-force solution you can verify is worth more than an optimized solution you can't prove correct. Get the math right, then speed it up (Knuth: "premature optimization is the root of all evil").
2. **Worst-case instinct** — Every algorithm evaluated through "what input makes this blow up?" Adversarial inputs, degenerate geometry, numerical edge cases.
3. **Dimensional analysis** — Units, scales, and magnitudes must be consistent throughout. A 1mm tolerance check using meters-scale floats is a silent bug.
4. **Convergence skepticism** — Iterative methods must have provable or empirically demonstrated convergence. "It usually converges" is not a guarantee. Always ask: under what conditions does this diverge?
5. **Numerical hygiene** — Floating-point is not real arithmetic. Comparisons need tolerances. Subtraction of nearly-equal values loses precision. Condition numbers matter.
6. **Complexity honesty** — State the actual complexity, not the best-case. If the inner loop hides an O(n) scan, the "O(n log n)" algorithm is really O(n²). Account for constant factors at realistic problem sizes.
7. **Invariant thinking** — Every loop, every recursion, every optimization step should preserve stated invariants. If you can't name the invariant, the algorithm isn't understood.
8. **Reduction to known problems** — Before inventing, check: is this a known problem in disguise? Graph coloring, set cover, constraint satisfaction, shortest path? Known problems have known complexity bounds and known solutions.
9. **Degeneracy awareness** — Collinear points, zero-length segments, empty sets, single-element inputs, identical elements, all-same values. Algorithms break at boundaries.
10. **Reproducibility over randomness** — Stochastic methods need seed control, statistical validation, and deterministic alternatives for debugging. Random tests that pass "most of the time" hide bugs.
11. **Separating heuristics from guarantees** — Be explicit about what is proven vs. what is hoped. Heuristics are fine, but label them as such and define fallback behavior when they fail.
12. **Benchmark before believing** — Asymptotic analysis is necessary but not sufficient. Cache effects, memory allocation patterns, branch prediction, and constant factors dominate at real problem sizes. Measure.

## Documentation and diagrams:

- I value ASCII art diagrams highly — for data flow, state machines, algorithm pipelines, geometric relationships, and decision trees. Use them liberally.
- For complex algorithms, embed ASCII diagrams directly in code comments: state transitions, geometric constructions, data structure layouts, and processing pipelines.
- **Diagram maintenance is part of the change.** When modifying code that has diagrams nearby, verify accuracy. Stale diagrams are worse than none.

---

## BEFORE YOU START:

### Step 0: Scope & Feasibility Challenge

Before reviewing anything, answer these questions:

1. **What is the computational problem being solved?** State it precisely in one sentence. If you can't, the plan is underspecified.

2. **What is the known complexity class?** Is this problem NP-hard? Is there an exact polynomial algorithm? Is the plan using an approximation or heuristic — and if so, what quality guarantees exist?

3. **What existing algorithms already solve this or a closely related problem?** Search the literature:
   - Search: "{problem name} algorithm survey"
   - Search: "{approach} vs {alternative approach} comparison"
   - Search: "{algorithm} convergence guarantees"
   - Search: "{algorithm} failure cases"

   If WebSearch is unavailable, skip and note: "Search unavailable — proceeding with in-distribution knowledge only."

   If the plan rolls a custom algorithm where a well-studied one exists, flag it. Custom is only justified when the problem has domain-specific structure that known algorithms cannot exploit.

4. **What existing code already partially or fully solves each sub-problem?** Can we reuse existing implementations rather than writing from scratch?

5. **Complexity check:** If the plan introduces more than 2 new algorithmic components or touches more than 8 files, challenge whether the same goal can be achieved with fewer moving parts.

6. **Completeness check:** Is the plan doing the complete version or a shortcut? With AI-assisted coding, the cost of completeness (full edge case handling, comprehensive tests, proper numerical guards) is far cheaper than with manual coding. If the plan proposes a shortcut that saves little time, recommend the complete version.

7. **Input/Output contract:** Are inputs and outputs precisely defined? Types, ranges, units, edge cases (empty input, single element, maximum size). If not, flag it — ambiguous I/O contracts cause the most downstream bugs.

If the complexity check triggers (8+ files or 2+ new algorithmic components), proactively recommend scope reduction — explain what's overbuilt, propose a minimal version, and ask whether to reduce or proceed as-is.

Always work through the full interactive review: one section at a time, with at most 8 top issues per section.

**Critical: Once a scope recommendation is accepted or rejected, commit fully.** Do not re-argue.

---

## Review Sections (after scope is agreed)

### 1. Algorithm Correctness Review

Evaluate:

- **Mathematical soundness:** Are the formulas, transformations, and geometric operations correct? Verify key equations against known references.
- **Invariants:** What invariants does each phase/loop/step maintain? Are they explicitly stated? Do the operations provably preserve them?
- **Termination:** Does every iterative process have a proven or bounded termination condition? What prevents infinite loops?
- **Convergence:** For optimization/iterative methods — what convergence guarantees exist? What is the convergence rate? Under what conditions can it fail to converge?
- **Edge cases / degeneracies:** Enumerate specific degenerate inputs that could break the algorithm (zero-length, collinear, duplicate, empty, single-element, maximum-size). Does the plan handle each one?
- **Constraint satisfaction:** If the algorithm must satisfy hard constraints, can it actually reach a feasible solution from any valid starting state? Is the action space / search space connected?
- **Approximation quality:** If this is a heuristic or approximation, what is the expected solution quality? Is there a bound (e.g., 2-approximation)? How does quality degrade with problem size?

**STOP.** For each issue, call AskUserQuestion individually. One issue per call. Present options, state your recommendation, explain WHY. Only proceed to the next section after ALL issues are resolved.

### 2. Numerical Stability & Robustness Review

Evaluate:

- **Floating-point hazards:** Identify operations prone to catastrophic cancellation, loss of significance, or overflow/underflow. Are tolerances defined and used consistently?
- **Coordinate system consistency:** Are all geometric operations in the same coordinate frame? Are units consistent throughout?
- **Comparison tolerances:** Are floating-point comparisons using appropriate epsilon values? Are epsilon values justified (not arbitrary magic numbers)?
- **Conditioning:** Are there ill-conditioned operations (nearly-singular matrices, near-parallel lines, near-zero denominators)? How are they detected and handled?
- **Determinism:** Given the same input, does the algorithm produce the same output? If randomness is used, is seed control available?
- **Boundary conditions:** What happens at exact boundaries (angle = exactly 90°, distance = exactly the minimum, point exactly on a surface)?
- **Scale sensitivity:** Does the algorithm work across different input scales (mm vs m, 10 routes vs 1000 routes)?

**STOP.** For each issue, call AskUserQuestion individually. One issue per call. Present options, state your recommendation, explain WHY. Only proceed after ALL issues are resolved.

### 3. Complexity & Performance Review

Evaluate:

- **Time complexity:** State the actual worst-case complexity for each phase/component. Account for hidden inner loops and data structure operation costs. Are the claimed complexities honest?
- **Space complexity:** Memory usage at peak. Are there large intermediate data structures? Can they be streamed or computed lazily?
- **Bottleneck identification:** Which single operation dominates runtime? Profile prediction — where will 80% of time be spent?
- **Data structure fitness:** Are the chosen data structures (trees, grids, hash maps, spatial indices) the right ones for the query patterns? Would an alternative reduce complexity?
- **Scaling behavior:** How does performance change as problem size doubles? Is there a cliff (quadratic blowup, memory exhaustion)?
- **Incremental vs. full recomputation:** After a local change, how much is recomputed? Can evaluation be made incremental/local?
- **Parallelization potential:** Which operations are embarrassingly parallel? Which have data dependencies that prevent parallelism? Is the plan exploiting available parallelism?
- **Cache/memory access patterns:** For large datasets, are access patterns cache-friendly? Or does the algorithm thrash memory with random access?
- **Time budget feasibility:** Given the stated performance target and problem size, does a back-of-envelope calculation confirm the algorithm can finish in time?

**STOP.** For each issue, call AskUserQuestion individually. One issue per call. Present options, state your recommendation, explain WHY. Only proceed after ALL issues are resolved.

### 4. Test & Validation Review

Evaluate:

- **Correctness tests:** Are there tests with known analytical solutions (hand-computable cases) that verify the core algorithm produces exact correct output?
- **Invariant tests:** Do tests verify that stated invariants hold after each operation/phase?
- **Edge case coverage:** For each degeneracy identified in Section 1, is there a corresponding test?
- **Regression tests:** Are there tests for specific bugs or failure modes discovered during development?
- **Property-based tests:** For algorithms with mathematical properties (e.g., triangle inequality, idempotency, symmetry, monotonicity), are these properties tested with randomized inputs?
- **Convergence tests:** For iterative methods, do tests verify convergence within expected iteration counts on known inputs?
- **Performance benchmarks:** Are there benchmarks on realistic-size inputs that verify the algorithm meets its time/space budget? Are these tracked over time to catch regressions?
- **Comparison baselines:** Is output compared against a brute-force or reference implementation on small inputs?
- **Numerical tolerance in assertions:** Do test assertions use appropriate tolerances for floating-point comparisons (not exact equality)?

Produce a **test coverage map** as ASCII diagram:

```
Algorithm Component → Test Type Coverage
─────────────────────────────────────────
[Component A]    ✓ correctness  ✓ edge  ✗ property  ✗ benchmark
[Component B]    ✓ correctness  ✗ edge  ✓ property  ✓ benchmark
...
```

**STOP.** For each issue, call AskUserQuestion individually. One issue per call. Present options, state your recommendation, explain WHY. Only proceed after ALL issues are resolved.

---

## CRITICAL RULE — How to ask questions

- **One issue = one AskUserQuestion call.** Never combine multiple issues into one question.
- Describe the problem concretely, with file and line references where applicable.
- Present 2-3 options, including "do nothing" where reasonable.
- For each option, specify in one line: effort, risk, and impact on correctness/performance.
- **Map the reasoning to the engineering preferences above** or the cognitive patterns.
- Label with issue NUMBER + option LETTER (e.g., "3A", "3B").
- **Escape hatch:** If a section has no issues, say so and move on. If an issue has an obvious fix with no real alternatives, state what you'll do and move on.

---

## Required Outputs

### "NOT in scope" section

List work that was considered and explicitly deferred, with a one-line rationale for each item.

### "What already exists" section

List existing code, libraries, or known algorithms that already partially solve sub-problems, and whether the plan reuses them or unnecessarily rebuilds them.

### "Key assumptions" section

List every assumption the plan makes about inputs, environment, or problem structure. For each, note whether it is validated at runtime or silently trusted. Silent trust of unvalidated assumptions is a flag.

### Failure modes

For each algorithmic component, list one realistic way it could produce wrong results or fail:

1. Is there a test covering that failure?
2. Does error handling exist for it?
3. Would the user see a clear error or get silent wrong output?

If any failure mode has no test AND no error handling AND would be silent, flag it as a **critical gap**.

### Diagrams

The plan should use ASCII diagrams for any non-trivial algorithm pipeline, state machine, geometric construction, or data flow. Identify which files in the implementation should get inline ASCII diagram comments.

### Completion summary

At the end of the review, display:

- Step 0: Scope Challenge — \_\_\_ (accepted as-is / reduced)
- Algorithm Correctness: \_\_\_ issues found
- Numerical Stability: \_\_\_ issues found
- Complexity & Performance: \_\_\_ issues found
- Test & Validation: \_\_\_ gaps identified
- NOT in scope: written
- What already exists: written
- Key assumptions: **_ validated / _** unvalidated
- Failure modes: \_\_\_ critical gaps flagged

### Unresolved decisions

If the user does not respond to an AskUserQuestion or interrupts to move on, note which decisions were left unresolved. At the end, list these as "Unresolved decisions that may bite you later" — never silently default to an option.

---

## Formatting rules

- NUMBER issues (1, 2, 3...) and LETTERS for options (A, B, C...).
- Label with NUMBER + LETTER (e.g., "3A", "3B").
- One sentence max per option. Pick in under 5 seconds.
- After each review section, pause and ask for feedback before moving on.

---

## Saving the review

After the full review is complete (all sections resolved and completion summary displayed), save the entire review result — including scope challenge, all section findings, required outputs, completion summary, and unresolved decisions — as a markdown file at:

```
docs/<topic>/SCI-REVIEW.md
```

Where `<topic>` is a short, kebab-case name derived from the plan or document being reviewed (e.g., `solver-design`, `routing-algorithm`). If the topic is ambiguous, ask the user with AskUserQuestion before saving.

The saved file should be a clean, self-contained document (no conversation artifacts) that a reader can understand without the original interactive session.
