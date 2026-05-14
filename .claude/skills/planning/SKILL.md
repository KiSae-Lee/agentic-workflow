---
name: planning
description: Use when you have a specifications or requirements for a multi-step task, before touching the codes or the documents. Produces plan.md, task.md, and next-step.md, and runs a test-design-review loop until the plan's test coverage is verified Ready.
---

# Planning

You are an IT project planner. Write comprehensive implementation plans assuming the engineer has zero context for our codebase. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain.

**Announce at start**: "I'm using the planning skill."

## Bootstrap: Read Rules Before Planning

Before drafting any artifacts, read the following rules files to inform your planning decisions. Each rule applies at a specific planning phase:

| Rule file | When it applies | How it shapes the plan |
|---|---|---|
| `.claude/skills/control-tower/rules/ARCHITECTURE.md` | Plan drafting — layer design, folder structure, DI strategy | Ensure tasks follow Clean Architecture layers, DIP with traits, DTO isolation, error translation |
| `.claude/skills/control-tower/rules/CONVENTIONS.md` | Plan drafting — commit strategy, naming, tooling choices | Apply git commit conventions, naming standards, cross-platform guidelines to task steps |
| `.claude/skills/control-tower/rules/OBSERVABILITY.md` | Plan drafting — logging/tracing tasks | Include structured logging, correlation IDs, log-level discipline in relevant tasks |
| `.claude/skills/control-tower/rules/DOCUMENTATION.md` | Plan drafting — doc tasks and code examples | Ensure code examples in the plan follow rustdoc/Python doc standards |
| `.claude/skills/control-tower/rules/AWARENESS.md` | Plan drafting — integration points, DB, proto, startup | Apply lessons learned: verify cross-service contracts, match DB schema to ORM, pin deps, proto fixups |
| `.claude/skills/control-tower/rules/KEEP_IN_MIND.md` | Plan drafting — interface design, test quality | Design deep modules, prefer dependency injection, mock only at system boundaries |
| `.claude/skills/control-tower/rules/TDD.md` | Test design coverage loop | Used by `test-design-review` skill to verify test completeness |
| `.claude/skills/subagent-driven-development/tech-debt-template.md` | 15% rule application — priority score formula, P0-P3 thresholds, schema | Used to interpret `spec/tech-debt.md` and scope debt repayment into the phase |

**Procedure:** At the start of planning, read all rule files listed above. If any file is missing, note it and continue — do not block on absent rules.

<HARD-GATE>
- Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until (a) you have presented a plan, (b) the test-design coverage loop has terminated with verdict `Ready`, and (c) the user has approved the final plan. This applies to EVERY project regardless of perceived simplicity.
</HARD-GATE>

## Bootstrap: Read Spec Files

Before drafting, read the design specs to understand what you're planning for:

1. **`spec/atdd.md`** — acceptance criteria. Identify unchecked items to scope this phase.
2. **`spec/overview.md`** — architecture, tech stack, cross-cutting concerns.
3. **`spec/tech-docs/use-cases.md`** — actor interactions and flows for the features in scope.
4. **`spec/flows.md`** — sequence diagrams, state diagrams for the features in scope.
5. **`spec/tech-docs/constraints.md`** — hard boundaries that limit implementation choices.
6. **`spec/architecture-decisions.md`** — ADRs that constrain how to build.
7. **`spec/tech-docs/non-functional-requirements.md`** — performance/security targets tasks must meet.
8. **`spec/glossary.md`** — use consistent terminology in plan and task descriptions.
9. **`spec/tech-docs/data-model.md`**, **`spec/tech-docs/api-design.md`**, **`spec/tech-docs/deployment.md`**, **`spec/tech-docs/ui/`** — read when relevant to the features in scope.
10. **`spec/tech-debt.md`** (if it exists) — open Active entries. Used for the 15% rule below. Read the Dashboard and at minimum the top-priority Active entries; skip if file does not exist (no existing debt).

You do NOT need to read every spec file in full. Read what's relevant to the features being planned in this phase.

## Phase Scoping

Each planning cycle covers a **subset** of unchecked atdd.md items — not the entire design — plus mandatory tech-debt repayment per the 15% rule. To determine this phase's scope:

1. **Read `spec/atdd.md`** — find all unchecked (`- [ ]`) acceptance criteria.
2. **If previous phases exist** — read the latest `spec/history/<phase>/next-step.md` for the recommended next scope.
3. **Group by Feature** — pick one or more related Features that form a coherent deliverable.
4. **Right-size the phase** — a phase should be completable in days, not weeks. If a Feature is too large, pick a subset of its user stories.
5. **Apply the 15% tech-debt rule** (see below) — include all `P0` debt and enough `P1` debt to occupy ~15% of phase capacity.
6. **Announce scope** — tell the user which atdd.md items + which `TD-NNN` entries this phase covers before drafting.

### 15% Tech-Debt Rule

Per `.claude/skills/subagent-driven-development/tech-debt-template.md` and SOP-A2-03:

1. **Read `spec/tech-debt.md`** (if it exists). Sort Active entries by **Priority score** descending.
2. **All `P0` entries are mandatory** — include them in this phase regardless of remaining capacity. P0 means score ≥ 50 — a P0 left unaddressed will keep producing rework.
3. **Add `P1` entries** in priority-score order until the cumulative Principal (MD) of selected debt reaches ~15% of the total estimated phase capacity. Stop at the first `P1` that would push you over 15% — unless skipping it leaves you below 10%, in which case include it.
4. **`P2` / `P3` entries are not mandatory.** Mention high-score P2 entries in the next-step.md if they fit a future phase well.
5. **If no `tech-debt.md` exists** or all entries are `P2`/`P3` with low scores — skip the rule. Note in `plan.md` that no debt repayment was scoped this phase and why.
6. **If P0 alone exceeds 25% of phase capacity** — flag to the user. The phase should either be re-scoped (drop a Feature) or split (debt-repayment-only phase). Don't silently ship a phase that's mostly debt.

Document the result in `plan.md` under "Phase scope" as a separate `Tech debt repayment` line listing `TD-NNN` IDs and total MD (e.g. `TD-003, TD-007 (5 MD ≈ 15% of 33 MD phase capacity)`). The corresponding tasks live in `task.md` like any other task.

## End-to-end flow

```mermaid
flowchart TD
    A[Read spec files + atdd.md + tech-debt.md] --> A1[Scope phase: unchecked atdd.md items + 15% rule TD-NNN]
    A1 --> B[Draft plan.md · task.md · next-step.md]
    B --> C[Invoke test-design-review skill on spec/history/<phase>/]
    C --> D[Read test-design-review.md]
    D --> E{Verdict == Ready?<br/>0 Critical · ≤ 2 Important}
    E -->|No| F[Revise task.md · plan.md per report]
    F --> C
    E -->|Yes| G[Final docs ready]
    G --> H[Execution handoff prompt]
```

The loop edits files **in place** under `spec/history/<phase>/`. The final docs are the iterated artifacts, not the first draft.

## Phase Directory Structure

Planning artifacts are bite-sized — each planning cycle covers one phase of the full-suite design. To preserve history across cycles, each phase gets its own subdirectory:

```
spec/
  common.md                      ← shared understanding (owned by grill-me)
  atdd.md                        ← acceptance criteria — the progress tracker
  glossary.md                    ← shared vocabulary (owned by design)
  overview.md                    ← architecture overview (owned by design)
  architecture-decisions.md      ← ADRs (owned by design)
  flows.md                       ← sequence/state/event diagrams (owned by design)
  tech-docs/                     ← deeper technical specs (owned by design)
    constraints.md
    non-functional-requirements.md
    use-cases.md
    ... (data-model.md, api-design.md, deployment.md, ui/ when applicable)
  arch-review/                   ← timestamped arch-review reports (owned by arch-review)
    YYYY-MM-DD HH:MM:SS.md
  sci-review/                    ← timestamped sci-review reports (owned by sci-review, when applicable)
    YYYY-MM-DD HH:MM:SS.md
  history/                       ← all planning + implementation phases (owned by planning + subagent-driven-development)
    2026-05-12T09-30-00_authentication/
      plan.md
      task.md
      next-step.md
      test-design-review.md
      code-review.md
      implementation-report.md
    2026-05-20T14-00-00_workspace-invitation/
      plan.md
      task.md
      ...
```

**Phase directory naming:** Use `YYYY-MM-DDTHH-MM-SS_keyword` format where the date+time is the current local time when the planning cycle begins and the keyword is a short kebab-case name derived from the phase scope (e.g., `2026-05-12T09-30-00_authentication`, `2026-05-20T14-00-00_workspace-invitation`). Generate the timestamp with `date "+%Y-%m-%dT%H-%M-%S"`. To find the latest phase, sort directories under `spec/history/` matching `[0-9]*` and pick the last one.

## Saving Artifacts

YOU MUST CREATE EXACTLY THREE SEPARATE FILES for each planning cycle. Save them under the current phase directory:

1. **`spec/history/<phase>/plan.md`**: The high-level plan, architecture, and tech stack for this phase.
2. **`spec/history/<phase>/task.md`**: The detailed, bite-sized implementation tasks for this phase.
3. **`spec/history/<phase>/next-step.md`**: Suggestion for the immediate next logical step or feature to work on after this phase is completed.

DO NOT combine them into a single file.

## File 1: Plan Guideline (`spec/history/<phase>/plan.md`)

**Every `plan.md` MUST start with this exact header structure, and MUST NOT contain the task list. It is ONLY for the high-level plan:**

```markdown
# [Goal/Feature Name] Implementation Plan

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

**Phase scope (from atdd.md):**
- Feature: [Feature name] → [User story 1], [User story 2]
- Feature: [Feature name] → [User story 3]

**Tech debt repayment (15% rule):**
- TD-NNN, TD-NNN (N MD ≈ N% of N MD phase capacity)
- _If none: "No tech debt scoped this phase — see plan section below for rationale."_

---
```

The phase scope section creates traceability from the plan back to the acceptance criteria it implements, and from the implementation back to the tech-debt ledger.

## File 2: Task Implementation (`spec/history/<phase>/task.md`)

**All tasks MUST go into `task.md`. Do NOT put tasks in `plan.md`.**

**Each step is one action (2-5 minutes):**

- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

**Every task inside `task.md` MUST use this exact structure:**

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

## File 3: Proposed Next Step (`spec/history/<phase>/next-step.md`)

**This file should contain a single, clear recommendation for what the user should work on next after completing the current plan. It helps maintain momentum.**

```markdown
# Proposed Next Step

**[Name of next feature/goal]**

[1-2 sentences explaining why this is the logical next step based on what was just built]
```

## Test Design Coverage Loop (mandatory before handoff)

Once `plan.md`, `task.md`, and `next-step.md` exist on disk, you must verify the plan's **test design completeness** against the project's TDD standard before handing off to implementation. Test gaps caught at the plan stage cost orders of magnitude less than gaps caught after code ships.

### Preconditions

- `.claude/skills/control-tower/rules/TDD.md` exists at the codebase root.
  - **If absent:** announce `"Test design coverage loop skipped — .claude/skills/control-tower/rules/TDD.md not found."` and proceed directly to Execution Handoff. Do not block planning on a missing standard.

### Loop procedure

```mermaid
flowchart TD
    Start([Docs saved]) --> Iter[Iteration N starts]
    Iter --> Invoke[Invoke test-design-review skill<br/>codebase = repo root<br/>target = spec/history/<phase>/<br/>output = spec/history/<phase>/test-design-review.md]
    Invoke --> Read[Read test-design-review.md<br/>Extract Verdict + gap counts + Top 3 actions]
    Read --> Decide{Verdict?}
    Decide -->|Ready<br/>0 Critical · ≤ 2 Important| Done([Exit loop · proceed to handoff])
    Decide -->|Needs fixes| Cap{Iteration < 3?}
    Decide -->|Blocked| Halt([Halt: surface blocker to user])
    Cap -->|Yes| Revise[Revise task.md / plan.md<br/>per Critical & Important recommendations]
    Cap -->|No| Escalate([Iteration cap hit · ask user how to proceed])
    Revise --> Iter
```

**Steps each iteration:**

1. Dispatch the `test-design-review` skill with:
   - **Codebase path**: the project root.
   - **Target items**: `spec/history/<phase>/plan.md` and `spec/history/<phase>/task.md` (and any associated source paths the plan touches).
   - **Output path** (override default): `spec/history/<phase>/test-design-review.md`.
2. Read the resulting `test-design-review.md`. Extract the **Verdict**, gap counts, and the **Top 3 actions** block.
3. Apply the verdict rule:
   - **`Ready`** (0 Critical, ≤ 2 Important) → exit the loop. Proceed to Execution Handoff.
   - **`Needs fixes`** → revise the plan and loop again.
   - **`Blocked`** → halt the loop. Surface the blocker to the user (typically a missing standard or unidentifiable target) and decide together how to proceed.
4. When revising, edit `task.md` first (most gaps are missing test tasks); touch `plan.md` only if the gap is architectural (e.g., a whole test category missing from the strategy section). Each Critical and Important recommendation must map to a concrete change — added test tasks, new boundary cases, mutation-testing setup tasks, security-test tasks, etc.

### Termination

- **Convergence:** verdict becomes `Ready`. The loop exits and you present the final docs.
- **Iteration cap:** after 3 iterations without convergence, stop and ask the user: _"After N iterations, the plan still has X Critical / Y Important gaps. Do you want to (a) accept the remaining gaps and proceed, (b) iterate further with my guidance on a specific gap, or (c) revisit the requirements?"_
- **Blocked:** the loop halts immediately. Do not attempt revisions until the blocker is resolved.

### What changes during revision

| Gap class     | Typical revision                                                                                                                                                                                   |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Critical**  | Add the missing required test category as new tasks in `task.md` (e.g., a security-test task for a payment flow), plus updates to the relevant section of `plan.md` if it's a structural omission. |
| **Important** | Add boundary / edge-case tasks, configure missing tooling (coverage, mutation), strengthen weak assertions in planned tests.                                                                       |
| **Minor**     | Optional. Track in `next-step.md` or note them in `plan.md` for follow-up; do not block on these.                                                                                                  |

The `test-design-review.md` report from the **final** iteration stays committed alongside the plan as a record that the gate was satisfied.

---

## Execution Handoff

After the test-design coverage loop has terminated with `Verdict: Ready` (or after the user explicitly accepted remaining gaps at the iteration cap):

Use `AskUserQuestion`:

- Question: "Plan complete and saved to `spec/history/<phase>/`. Ready to start implementation?"
- Header: "Next step"
- Options:
  1. **"Start subagent-driven development" (Recommended)** — Invoke the `subagent-driven-development` skill to begin implementation immediately.
  2. **"Not now"** — End here; the user will start implementation later.

**If the user selects "Start subagent-driven development":**

- **REQUIRED SUB-SKILL:** Use subagent-driven-development
- Stay in this session
- Fresh subagent per task + code review

> **Note:** Sci-review runs during the Design stage (auto-invoked by the `design` skill) before planning begins. It is not part of the planning handoff.
