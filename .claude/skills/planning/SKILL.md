---
name: planning
description: Use when you have a specifications or requirements for a multi-step task, before touching the codes or the documents. Produces PLAN.md, TASK.md, and NEXT_STEP.md, and runs a test-design-review loop until the plan's test coverage is verified Ready.
---

# Planning

You are an IT project planner. Write comprehensive implementation plans assuming the engineer has zero context for our codebase. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain.

**Announce at start**: "I'm using the planning skill."

<HARD-GATE>
- Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until (a) you have presented a plan, (b) the test-design coverage loop has terminated with verdict `Ready`, and (c) the user has approved the final plan. This applies to EVERY project regardless of perceived simplicity.
</HARD-GATE>

## End-to-end flow

```mermaid
flowchart TD
    A[Receive specifications] --> B[Draft PLAN.md · TASK.md · NEXT_STEP.md]
    B --> C[Invoke test-design-review skill on docs/&lt;topic&gt;/]
    C --> D[Read TEST-DESIGN-REVIEW.md]
    D --> E{Verdict == Ready?<br/>0 Critical · ≤ 2 Important}
    E -->|No| F[Revise TASK.md · PLAN.md per report]
    F --> C
    E -->|Yes| G[Final docs ready]
    G --> H[Execution handoff prompt]
```

The loop edits files **in place** under `docs/<topic>/`. The final docs are the iterated artifacts, not the first draft.

## Saving Artifacts

YOU MUST CREATE EXACTLY THREE SEPARATE FILES for each goal/feature unless specified otherwise by the project's `CLAUDE.md`, `AGENT.md` or `GEMINI.md` etc. Create a dedicated folder for the specific goal or feature:

1. **`docs/<topic>/PLAN.md`**: The high-level plan, architecture, and tech stack.
2. **`docs/<topic>/TASK.md`**: The detailed, bite-sized implementation tasks.
3. **`docs/<topic>/NEXT_STEP.md`**: Suggestion for the immediate next logical step or feature to work on after this goal is completed.

DO NOT combine them into a single file.

## File 1: Plan Guideline (`docs/<topic>/PLAN.md`)

**Every `PLAN.md` MUST start with this exact header structure, and MUST NOT contain the task list. It is ONLY for the high-level plan:**

```markdown
# [Goal/Feature Name] Implementation Plan

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## File 2: Task Implementation (`docs/<topic>/TASK.md`)

**All tasks MUST go into `TASK.md`. Do NOT put tasks in `PLAN.md`.**

**Each step is one action (2-5 minutes):**

- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

**Every task inside `TASK.md` MUST use this exact structure:**

### TDD Tasks (pure logic, domain, application layers)

Use TDD when the task involves testable logic — state reducers, data transformations, domain models, utility functions.

````markdown
### Task N: [Component Name]

**Files:**

- Create: `exact/path/to/file.rs`
- Modify: `exact/path/to/existing.rs:123-145`
- Test: `tests/exact/path/to/test.rs`

**Step 1: Write the failing test**

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_specific_behavior() {
        let result = function(input);
        assert_eq!(result, expected);
    }
}
```

**Step 2: Run test to verify it fails**

Run: `cargo test test_specific_behavior -- --nocapture`
Expected: FAIL with "cannot find function"

**Step 3: Write minimal implementation**

```rust
pub fn function(input: InputType) -> OutputType {
    expected
}
```

**Step 4: Run test to verify it passes**

Run: `cargo test test_specific_behavior -- --nocapture`
Expected: PASS
````

### Non-TDD Tasks (GPU, shaders, visual output, pipeline setup)

Some tasks are impractical to TDD — shader compilation, render pipeline wiring, visual output verification, wgpu resource setup. For these, use a build-and-verify approach instead.

````markdown
### Task N: [Component Name]

**Files:**

- Create: `exact/path/to/file.rs`
- Modify: `exact/path/to/existing.rs:123-145`

**Step 1: Implement the component**

```rust
// Complete implementation code
```

**Step 2: Verify it compiles**

Run: `cargo build`
Expected: compiles without errors

**Step 3: Verify visually / integration**

Run: `cargo run --example native_viewer`
Expected: [describe what should appear or what behavior to confirm]
````

## File 3: Proposed Next Step (`docs/<topic>/NEXT_STEP.md`)

**This file should contain a single, clear recommendation for what the user should work on next after completing the current plan. It helps maintain momentum.**

```markdown
# Proposed Next Step

**[Name of next feature/goal]**

[1-2 sentences explaining why this is the logical next step based on what was just built]
```

## Test Design Coverage Loop (mandatory before handoff)

Once `PLAN.md`, `TASK.md`, and `NEXT_STEP.md` exist on disk, you must verify the plan's **test design completeness** against the project's TDD standard before handing off to implementation. Test gaps caught at the plan stage cost orders of magnitude less than gaps caught after code ships.

### Preconditions

- `.claude/rules/TDD.md` exists at the codebase root.
  - **If absent:** announce `"Test design coverage loop skipped — .claude/rules/TDD.md not found."` and proceed directly to Execution Handoff. Do not block planning on a missing standard.

### Loop procedure

```mermaid
flowchart TD
    Start([Docs saved]) --> Iter[Iteration N starts]
    Iter --> Invoke[Invoke test-design-review skill<br/>codebase = repo root<br/>target = docs/&lt;topic&gt;/<br/>output = docs/&lt;topic&gt;/TEST-DESIGN-REVIEW.md]
    Invoke --> Read[Read TEST-DESIGN-REVIEW.md<br/>Extract Verdict + gap counts + Top 3 actions]
    Read --> Decide{Verdict?}
    Decide -->|Ready<br/>0 Critical · ≤ 2 Important| Done([Exit loop · proceed to handoff])
    Decide -->|Needs fixes| Cap{Iteration < 3?}
    Decide -->|Blocked| Halt([Halt: surface blocker to user])
    Cap -->|Yes| Revise[Revise TASK.md / PLAN.md<br/>per Critical & Important recommendations]
    Cap -->|No| Escalate([Iteration cap hit · ask user how to proceed])
    Revise --> Iter
```

**Steps each iteration:**

1. Dispatch the `test-design-review` skill with:
   - **Codebase path**: the project root.
   - **Target items**: `docs/<topic>/PLAN.md` and `docs/<topic>/TASK.md` (and any associated source paths the plan touches).
   - **Output path** (override default): `docs/<topic>/TEST-DESIGN-REVIEW.md`.
2. Read the resulting `TEST-DESIGN-REVIEW.md`. Extract the **Verdict**, gap counts, and the **Top 3 actions** block.
3. Apply the verdict rule:
   - **`Ready`** (0 Critical, ≤ 2 Important) → exit the loop. Proceed to Execution Handoff.
   - **`Needs fixes`** → revise the plan and loop again.
   - **`Blocked`** → halt the loop. Surface the blocker to the user (typically a missing standard or unidentifiable target) and decide together how to proceed.
4. When revising, edit `TASK.md` first (most gaps are missing test tasks); touch `PLAN.md` only if the gap is architectural (e.g., a whole test category missing from the strategy section). Each Critical and Important recommendation must map to a concrete change — added test tasks, new boundary cases, mutation-testing setup tasks, security-test tasks, etc.

### Termination

- **Convergence:** verdict becomes `Ready`. The loop exits and you present the final docs.
- **Iteration cap:** after 3 iterations without convergence, stop and ask the user: *"After N iterations, the plan still has X Critical / Y Important gaps. Do you want to (a) accept the remaining gaps and proceed, (b) iterate further with my guidance on a specific gap, or (c) revisit the requirements?"*
- **Blocked:** the loop halts immediately. Do not attempt revisions until the blocker is resolved.

### What changes during revision

| Gap class | Typical revision |
|---|---|
| **Critical** | Add the missing required test category as new tasks in `TASK.md` (e.g., a security-test task for a payment flow), plus updates to the relevant section of `PLAN.md` if it's a structural omission. |
| **Important** | Add boundary / edge-case tasks, configure missing tooling (coverage, mutation), strengthen weak assertions in planned tests. |
| **Minor** | Optional. Track in `NEXT_STEP.md` or note them in `PLAN.md` for follow-up; do not block on these. |

The `TEST-DESIGN-REVIEW.md` report from the **final** iteration stays committed alongside the plan as a record that the gate was satisfied.

---

## Execution Handoff

After the test-design coverage loop has terminated with `Verdict: Ready` (or after the user explicitly accepted remaining gaps at the iteration cap):

### Sci-Review Detection

Before presenting options, scan the plan for algorithmic/scientific content. Check if the plan involves ANY of:

- Algorithm design or implementation (sorting, searching, graph algorithms, optimization)
- Numerical methods (floating-point arithmetic, interpolation, solvers, convergence)
- Scientific computing (physics simulation, geometry processing, mesh operations)
- AI/ML components (training loops, loss functions, data pipelines)
- Mathematical transformations (matrix operations, coordinate systems, projections)
- Performance-critical algorithms where complexity analysis matters

**If algorithmic/scientific content is detected**, use `AskUserQuestion`:

- Question: "Plan complete and saved to `docs/<topic>/`. This plan involves algorithmic/scientific content — a sci-review can catch correctness, numerical stability, and complexity issues before implementation. How would you like to proceed?"
- Header: "Next step"
- Options:
  1. **"Run sci-review first" (Recommended for algorithmic plans)** — Invoke the `sci-review` skill to review the plan before implementation.
  2. **"Start subagent-driven development"** — Skip review, begin implementation immediately.
  3. **"Not now"** — End here; the user will start later.

**If NO algorithmic/scientific content**, use `AskUserQuestion`:

- Question: "Plan complete and saved to `docs/<topic>/`. Ready to start implementation?"
- Header: "Next step"
- Options:
  1. **"Start subagent-driven development" (Recommended)** — Invoke the `subagent-driven-development` skill to begin implementation immediately.
  2. **"Not now"** — End here; the user will start implementation later.

**If the user selects "Run sci-review first":**

- **REQUIRED SUB-SKILL:** Use sci-review
- After sci-review completes, ask again whether to start subagent-driven development

**If the user selects "Start subagent-driven development":**

- **REQUIRED SUB-SKILL:** Use subagent-driven-development
- Stay in this session
- Fresh subagent per task + code review
