---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks in the current session
---

# Subagent-Driven Development

Execute plan by dispatching fresh subagent per task, with two-stage review after each: spec compliance review first, then code quality review.

**Core principle:** Fresh subagent per task + two-stage review (spec then quality) = high quality, fast iteration

<HARD-GATE>
This skill is NOT complete until ALL THREE output artifacts exist on disk:
1. `spec/history/<phase>/code-review.md` — produced by dispatching the `code-review` skill
2. `spec/history/<phase>/implementation-report.md` — written by you after code review
3. `spec/tech-debt.md` — created or updated (Dashboard + Active + Closed sections recomputed for this phase). If no debt was registered or closed in this phase AND the file already exists, only the Dashboard timestamp + Inflow/Outflow=0 need updating. If the file does not exist and no debt was incurred or discovered, you may skip this artifact and note "no tech debt registered" in the implementation report.

After all implementation tasks finish, you MUST produce these artifacts before reporting completion to the user or returning control to control-tower. If the user interrupts or changes direction mid-execution, resume artifact production when control returns. No exceptions.
</HARD-GATE>

## Bootstrap: Read Rules + Templates

Before dispatching implementer subagents, read these rules and templates and include relevant sections in their prompts:

| File | Why |
|---|---|
| `.claude/skills/control-tower/rules/ARCHITECTURE.md` | Clean Architecture layers implementers must follow |
| `.claude/skills/control-tower/rules/CONVENTIONS.md` | Git, naming, and tooling conventions for commits and code |
| `.claude/skills/control-tower/rules/DOCUMENTATION.md` | Doc standards for code the implementers write |
| `.claude/skills/control-tower/rules/OBSERVABILITY.md` | Logging and tracing standards for production code |
| `.claude/skills/control-tower/rules/KEEP_IN_MIND.md` | Interface design, testability, mocking boundaries |
| `.claude/skills/control-tower/rules/TDD.md` | TDD methodology bundled into implementer prompts |
| `.claude/skills/control-tower/rules/AWARENESS.md` | Integration lessons to avoid known pitfalls |
| `./tech-debt-template.md` | Schema + lifecycle for `spec/tech-debt.md` — bundle into implementer prompt so any new `TODO`/`FIXME`/`HACK` is registered with a `TD-NNN` ID |

**Procedure:** Read all listed files. Pass relevant content to implementer and reviewer subagents as context.

## Bootstrap: Read Existing Tech-Debt State

Before dispatching the first implementer, read `spec/tech-debt.md` if it exists. Note:
- The next available `TD-NNN` ID (max existing + 1) — see ID allocation rule below.
- Open entries adjacent to files this phase will touch (Boy Scout candidates) — bundle into prompts for tasks in those areas.
- Any P0 entries — they must be addressed in this phase regardless of phase scope (planning's 15% rule already accounts for P0/P1, but verify).

If `spec/tech-debt.md` does not exist, start ID counter at `TD-001`. The file is created lazily on first registration.

### TD-NNN ID Allocation Under Parallel Execution

Implementers do NOT modify `spec/tech-debt.md` directly — they emit entries in their task reports, and you (the dispatcher) write the file at phase finalization. So there are no file-level merge conflicts. The remaining hazard is **ID collisions in code annotations**: two parallel implementers might both pick `TD-040` and stamp it into different files, producing two TODOs that reference the same ID.

**Allocation rule:** Reserve a non-overlapping range of 10 IDs per parallel implementer in a wave, in dispatch order.

Example — three implementers in Wave 1, current max in `spec/tech-debt.md` is `TD-039`:
- Implementer A → may use `TD-040` through `TD-049`
- Implementer B → may use `TD-050` through `TD-059`
- Implementer C → may use `TD-060` through `TD-069`

Inject the assigned range into each implementer prompt's `## Tech Debt Context` block: `Assigned ID range: TD-040 through TD-049`. The implementer prompt's instructions tell them to start at the lowest and increment within the range only.

Between waves, compute the next starting ID as `max(current_file, highest_ID_reported_in_completed_waves) + 1`. Gaps from unused IDs in a range are fine — IDs are never reused, but skipping is allowed.

Sequential implementers can be given a single starting ID with no upper bound (e.g. `Assigned ID range: TD-040 onward`), since each completes before the next starts.

## When to Use

- You have an implementation plan (from planning skill)
- Tasks are mostly independent
- You want to stay in this session

## The Process

1. **Read plan** — Find the current phase directory (sort `spec/history/[0-9]*/` directories, pick the last one — phase folders use the `YYYY-MM-DDTHH-MM-SS_keyword` naming convention) and extract all tasks with full text from its `task.md`, note context
2. **Create task tracking** — Use `TaskCreate` for each task
3. **Analyze dependencies** — Classify tasks as independent or dependent:
   - **Independent:** Tasks that touch different files/modules with no shared state
   - **Dependent:** Tasks where one requires output from another, or they modify the same files
4. **Execute tasks** — Use the appropriate strategy:

### Independent Tasks → Parallel Execution

Dispatch independent tasks simultaneously using `isolation: "worktree"` so each subagent works on an isolated copy of the repo. This prevents git conflicts and file contention.

For each independent task in the batch:
   a. Update task status to `in_progress` via `TaskUpdate`
   b. Dispatch implementer subagent with `isolation: "worktree"` (see `./implementer-prompt.md`)
   c. Run in background — move to next independent task immediately

After all parallel tasks complete:
   d. For each completed task, dispatch spec reviewer (see `./spec-reviewer-prompt.md`)
   e. If spec issues found → dispatch fix subagent in the same worktree → re-review until ✅
   f. Dispatch code quality reviewer (see `./code-quality-reviewer-prompt.md`)
   g. If quality issues found → dispatch fix subagent in the same worktree → re-review until ✅
   h. Merge the worktree branch into the main branch: `git merge <worktree-branch>`
   i. Mark task complete via `TaskUpdate`

Resolve any merge conflicts before proceeding to the next batch.

### Dependent Tasks → Sequential Execution

For tasks that depend on previous tasks' output:
   a. Update task status to `in_progress` via `TaskUpdate`
   b. Dispatch implementer subagent (see `./implementer-prompt.md`)
   c. If implementer asks questions → answer, re-dispatch
   d. Implementer implements, tests, commits, self-reviews
   e. Dispatch spec reviewer subagent (see `./spec-reviewer-prompt.md`)
   f. If spec issues found → implementer fixes → re-review until ✅
   g. Dispatch code quality reviewer subagent (see `./code-quality-reviewer-prompt.md`)
   h. If quality issues found → implementer fixes → re-review until ✅
   i. Mark task complete via `TaskUpdate`

### Mixed Plans

When a plan has both independent and dependent tasks, group them into execution waves:

```
Wave 1 (parallel): Task 1, Task 2, Task 3  ← all independent
Wave 2 (sequential): Task 4                ← depends on Task 1
Wave 3 (parallel): Task 5, Task 6          ← independent, but depend on Task 4
```

Each wave completes (including reviews and merges) before the next wave begins.

5. **Update atdd.md** — Read `spec/atdd.md` and mark completed items as `[x]`. Add any new items discovered during implementation. Do NOT remove deferred items.
6. **Write `spec/tech-debt.md`** — You (the dispatcher) are the sole writer of this file; implementers never touch it. After all worktrees have merged:
   a. **Create** the file from `./tech-debt-template.md` skeleton if it does not exist.
   b. **Append entries reported as Opened** by each implementer (their report includes the full schema text verbatim).
   c. **Move entries reported as Closed** from `## Active` to `## Closed`, prepending the closed-fields block (Closed in phase, Closed at, Commit / PR, Outcome — fill from the implementer report; commit hash also verifiable via `git log`).
   d. **Recompute the Dashboard** — Last updated timestamp + phase folder, Open count, Inflow/Outflow, Ratio, Top 5 (per `./tech-debt-template.md` "Update Rules").
   Skip steps (a)–(c) only if no implementer reported any Opened or Closed entries this phase. Always perform step (d) so the timestamp advances.
7. **Final review** — Dispatch code-review skill for the entire implementation (report saved to `spec/history/<phase>/code-review.md`). The code-review skill enforces DoD: every new `TODO`/`FIXME`/`HACK` must reference an existing `TD-NNN`, and `*-deliberate` debt must have a populated entry.
8. **Implementation report** — Write `spec/history/<phase>/implementation-report.md` summarizing what was built, files changed, **TDs opened / closed / deferred this phase**, and any open concerns. If deliberate debt was incurred, include a `## Deliberate Debt` section with the 5 fields per SOP-A2-07:
   - **Reason** — taken verbatim from the implementer's reported TD entry; must be concrete.
   - **Repayment target** — taken from the TD entry.
   - **Repayment TD-NNN** — the TD-NNN of the entry.
   - **Owner** — the implementer (or named contributor responsible for repayment).
   - **Approver** — the human user. NEVER list yourself (the dispatcher) or the implementer as approver. If running fully autonomously without a user available, prompt for explicit approval before recording deliberate debt.

## Prompt Templates

- `./implementer-prompt.md` - Dispatch implementer subagent
- `./spec-reviewer-prompt.md` - Dispatch spec compliance reviewer subagent
- `./code-quality-reviewer-prompt.md` - Dispatch code quality reviewer subagent

## Output Artifacts

At the end of execution, these files must exist (or be intentionally skipped per the rules below):

- **`spec/history/<phase>/code-review.md`** — Final code review report
- **`spec/history/<phase>/implementation-report.md`** — Summary of implementation: what was built, files changed, test results, TDs opened/closed/deferred this phase, open concerns
- **`spec/tech-debt.md`** — Updated dashboard + Active/Closed sections. May be absent only if no debt has ever been registered in the project AND none was registered this phase.

## Example Workflow

```
You: I'm using Subagent-Driven Development to execute this plan.

[Read task.md: extract all 5 tasks with full text and context]
[TaskCreate for each task]
[Analyze dependencies:
  Task 1 (buffer module) — independent
  Task 2 (pipeline state) — independent
  Task 3 (render pass)   — depends on Task 1 & 2
  Task 4 (picking)       — independent
  Task 5 (integration)   — depends on Task 3 & 4
]

--- Wave 1 (parallel): Tasks 1, 2, 4 ---

[TaskUpdate: Tasks 1, 2, 4 in_progress]
[Dispatch Task 1 implementer with isolation: "worktree", run_in_background]
[Dispatch Task 2 implementer with isolation: "worktree", run_in_background]
[Dispatch Task 4 implementer with isolation: "worktree", run_in_background]

[All three complete]

[Spec review Task 1] ✅
[Code quality review Task 1] ✅
[Merge Task 1 worktree branch]
[TaskUpdate: Task 1 completed]

[Spec review Task 2] ✅
[Code quality review Task 2] ❌ — magic number
[Fix subagent in Task 2 worktree] → extracted constant
[Code quality re-review Task 2] ✅
[Merge Task 2 worktree branch]
[TaskUpdate: Task 2 completed]

[Spec review Task 4] ✅
[Code quality review Task 4] ✅
[Merge Task 4 worktree branch]
[TaskUpdate: Task 4 completed]

--- Wave 2 (sequential): Task 3 ---

[TaskUpdate: Task 3 in_progress]
[Dispatch Task 3 implementer — depends on merged Task 1 & 2]
[Spec review] ✅  [Code quality review] ✅
[TaskUpdate: Task 3 completed]

--- Wave 3 (sequential): Task 5 ---

[TaskUpdate: Task 5 in_progress]
[Dispatch Task 5 implementer — depends on Task 3 & 4]
[Spec review] ✅  [Code quality review] ✅
[TaskUpdate: Task 5 completed]

--- Final ---

[Update atdd.md — mark completed items, add discovered items]
[Update spec/tech-debt.md — append TD-NNN entries opened this phase, move closed ones, recompute Dashboard]
[Dispatch code-review skill for entire implementation → code-review.md]
[Write implementation-report.md including TDs opened/closed/deferred + Deliberate Debt section if any]

Done!
```

## Terminal State

<HARD-GATE>
Before announcing completion or returning control to the caller, verify ALL THREE artifacts:

```
spec/history/<phase>/code-review.md           — exists? → yes/no
spec/history/<phase>/implementation-report.md — exists? → yes/no
spec/tech-debt.md                              — current? → yes (Dashboard timestamp = current phase folder) / N/A (no debt ever registered)
```

If any are missing or stale, produce/refresh them NOW. Do not skip because:
- The user said "don't commit" (artifacts are docs, not commits)
- The user interrupted mid-flow (resume artifact production)
- All tasks are marked complete (task completion ≠ skill completion)
- You ran out of things to implement (the final review IS part of implementation)
- "Nothing changed in tech-debt this phase" — the Dashboard timestamp must still advance
</HARD-GATE>

After BOTH artifacts exist, the workflow continues with:

1. **Update codebase-map.md** — reflect the new code structure
2. **Update atdd.md** — mark completed items, add any new items discovered
3. **Create/update summary.md** — write or update `spec/summary.md` following the template in `.claude/skills/subagent-driven-development/summary-template.md`

These updates happen in the main conversation, not via subagent. Subagent-driven-development ends after producing its output artifacts.

## Red Flags

**Never:**

- **End the skill without producing code-review.md and implementation-report.md** (the skill is not done until both exist)
- Start implementation on main/master branch without explicit user consent
- Skip reviews (spec compliance OR code quality)
- Proceed with unfixed issues
- Run parallel tasks without `isolation: "worktree"` (will cause git/file conflicts)
- Run dependent tasks in parallel (wrong execution order)
- Make subagent read plan file (provide full text instead)
- Skip scene-setting context (subagent needs to understand where task fits)
- Ignore subagent questions (answer before letting them proceed)
- **Start code quality review before spec compliance is ✅** (wrong order)
- Move to next task while either review has open issues

**If subagent asks questions:**

- Answer clearly and completely
- Provide additional context if needed
- Don't rush them into implementation

**If reviewer finds issues:**

- Implementer (same subagent) fixes them
- Reviewer reviews again
- Repeat until approved

**If subagent fails task:**

- Dispatch fix subagent with specific instructions
- Don't try to fix manually (context pollution)

## Integration

**Required workflow skills:**

- **planning** - Creates the plan this skill executes
- **code-review** - Dispatches code-reviewer agent for quality reviews
- **git-commit** - Use this skill when you create a commit

**Subagents receive:**

- **TDD methodology** - Bundled directly in the implementer prompt template
