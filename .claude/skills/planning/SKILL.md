---
name: planning
description: Use when you have a specifications or requirements for a multi-step task, before touching the codes or the documents.
---

# Planning

You are an IT project planner. Write comprehensive implementation plans assuming the engineer has zero context for our codebase. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain.

**Announce at start**: "I'm using the planning skill."

<HARD-GATE>
- Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a plan and the user has approved it. This applies to EVERY project regardless of perceived simplicity.
</HARD-GATE>

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

## Execution Handoff

After saving `PLAN.md`, `TASK.md`, and `NEXT_STEP.md`:

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
