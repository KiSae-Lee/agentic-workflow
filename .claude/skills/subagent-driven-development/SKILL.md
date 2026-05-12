---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks in the current session
---

# Subagent-Driven Development

Execute plan by dispatching fresh subagent per task, with two-stage review after each: spec compliance review first, then code quality review.

**Core principle:** Fresh subagent per task + two-stage review (spec then quality) = high quality, fast iteration

<HARD-GATE>
This skill is NOT complete until BOTH output artifacts exist on disk:
1. `spec/YYYYMMDD-keyword/code-review.md` — produced by dispatching the `code-review` skill
2. `spec/YYYYMMDD-keyword/implementation-report.md` — written by you after code review

After all implementation tasks finish, you MUST produce these artifacts before reporting completion to the user or returning control to control-tower. If the user interrupts or changes direction mid-execution, resume artifact production when control returns. No exceptions.
</HARD-GATE>

## Bootstrap: Read Rules

Before dispatching implementer subagents, read these rules and include relevant sections in their prompts:

| Rule file | Why |
|---|---|
| `.claude/skills/control-tower/rules/ARCHITECTURE.md` | Clean Architecture layers implementers must follow |
| `.claude/skills/control-tower/rules/CONVENTIONS.md` | Git, naming, and tooling conventions for commits and code |
| `.claude/skills/control-tower/rules/DOCUMENTATION.md` | Doc standards for code the implementers write |
| `.claude/skills/control-tower/rules/OBSERVABILITY.md` | Logging and tracing standards for production code |
| `.claude/skills/control-tower/rules/KEEP_IN_MIND.md` | Interface design, testability, mocking boundaries |
| `.claude/skills/control-tower/rules/TDD.md` | TDD methodology bundled into implementer prompts |
| `.claude/skills/control-tower/rules/AWARENESS.md` | Integration lessons to avoid known pitfalls |

**Procedure:** Read all listed rule files. Pass relevant content to implementer and reviewer subagents as context.

## When to Use

- You have an implementation plan (from planning skill)
- Tasks are mostly independent
- You want to stay in this session

## The Process

1. **Read plan** — Find the current phase directory (sort `spec/[0-9]*-*/` directories, pick the last one) and extract all tasks with full text from its `task.md`, note context
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
6. **Final review** — Dispatch code-review skill for the entire implementation (report saved to `spec/YYYYMMDD-keyword/code-review.md`)
7. **Implementation report** — Write `spec/YYYYMMDD-keyword/implementation-report.md` summarizing what was built, files changed, and any open concerns

## Prompt Templates

- `./implementer-prompt.md` - Dispatch implementer subagent
- `./spec-reviewer-prompt.md` - Dispatch spec compliance reviewer subagent
- `./code-quality-reviewer-prompt.md` - Dispatch code quality reviewer subagent

## Output Artifacts

At the end of execution, these files must exist:

- **`spec/YYYYMMDD-keyword/code-review.md`** — Final code review report
- **`spec/YYYYMMDD-keyword/implementation-report.md`** — Summary of implementation: what was built, files changed, test results, open concerns

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
[Dispatch code-review skill for entire implementation → code-review.md]
[Write implementation-report.md]

Done!
```

## Terminal State

<HARD-GATE>
Before announcing completion or returning control to the caller, verify BOTH artifacts exist:

```
spec/YYYYMMDD-keyword/code-review.md          — exists? → yes/no
spec/YYYYMMDD-keyword/implementation-report.md — exists? → yes/no
```

If either is missing, produce it NOW. Do not skip because:
- The user said "don't commit" (artifacts are docs, not commits)
- The user interrupted mid-flow (resume artifact production)
- All tasks are marked complete (task completion ≠ skill completion)
- You ran out of things to implement (the final review IS part of implementation)
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
