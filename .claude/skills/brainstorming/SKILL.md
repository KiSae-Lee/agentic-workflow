---
name: brainstorming
description: "Use after grill-me has produced COMMON.md, or when the user already has a clear spec. Reads COMMON.md and produces a full-suite DESIGN.md (not MVP-scoped). Explores approaches, drafts the design, then auto-invokes arch-review to validate and update DESIGN.md before transitioning to planning."
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - Write
  - Edit
  - AskUserQuestion
  - WebSearch
---

# Brainstorming — Ideas Into Full-Suite Designs

## Overview

Turn shared understanding (COMMON.md) into a fully formed, full-suite design. This is NOT MVP-scoped — design the complete system, then mark what belongs to MVP vs. later phases.

**Announce at start**: "I'm using the brainstorming skill."

<HARD-GATE>
- Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it.
- The design MUST be full-suite oriented. Mark MVP scope within the design, but design the whole system.
</HARD-GATE>

## Bootstrap: Read Rules

Before drafting any design, read these rules to inform your decisions:

| Rule file | Why |
|---|---|
| `.claude/skills/control-tower/rules/ARCHITECTURE.md` | Clean Architecture layers, folder structure, DIP strategy |
| `.claude/skills/control-tower/rules/CONVENTIONS.md` | Naming, git, markdown conventions |
| `.claude/skills/control-tower/rules/DOCUMENTATION.md` | Doc standards for code examples in the design |
| `.claude/skills/control-tower/rules/KEEP_IN_MIND.md` | Deep modules, interface design, testability |
| `.claude/skills/control-tower/rules/AWARENESS.md` | Cross-service contracts, DB schema, dependency lessons |
| `.claude/skills/control-tower/rules/OBSERVABILITY.md` | Logging and tracing design considerations |

**Procedure:** Read all listed rule files at the start. If any file is missing, note it and continue.

## Fast Path

If the user already has a clear spec, a detailed description, or says something like "I know what I want, just help me structure it" — skip reading COMMON.md and go straight to proposing approaches. Ask one confirmation question to validate your understanding, then move on.

## Checklist

You MUST complete these steps in order:

1. **Read COMMON.md** — load the shared understanding from `docs/<topic>/COMMON.md`
2. **Propose 2-3 approaches** — with trade-offs and your recommendation
3. **Draft design** — write a complete full-suite `DESIGN.md`
4. **Polish** — invoke writing skill to improve clarity and conciseness
5. **Present and revise** — present to user, incorporate feedback, iterate until approved
6. **Create TODO.md** — extract actionable items from approved design, mark MVP vs. later phases
7. **Auto-invoke arch-review** — arch-review validates and auto-updates DESIGN.md
8. **Transition to planning** — invoke planning skill to create implementation plan

Determine the design file path from context: use `docs/<topic>/DESIGN.md`. The `<topic>` should match the COMMON.md location. If the path is ambiguous, ask the user with `AskUserQuestion` before proceeding.

## The Process

<HARD-GATE>
EVERY question to the user MUST use the `AskUserQuestion` tool. Do NOT output a question as plain text and move on — you will not receive a response. Plain-text questions are invisible to the user in skill mode. The ONLY way to get user input is through `AskUserQuestion`.
</HARD-GATE>

**Loading shared understanding:**

- Read `docs/<topic>/COMMON.md` produced by grill-me
- If COMMON.md doesn't exist, fall back to exploring project context and asking clarifying questions (one at a time via `AskUserQuestion`)
- Internalize: problem statement, key decisions, constraints, success criteria, assumptions

**Exploring approaches:**

- Propose 2-3 different approaches using `AskUserQuestion` with `options` and `preview` fields
- Use `preview` to show concrete examples (architecture sketches, code snippets, data flow diagrams)
- Put your recommended option first with "(Recommended)" appended to its label
- Include trade-offs in each option's `description`

**Presenting the design:**

Write the complete design into the `DESIGN.md` file, covering relevant sections. Scale each section to its complexity: a few sentences if straightforward, up to 200-300 words if nuanced.

The design MUST include:

- **Full-suite scope** — the complete system design
- **MVP marker** — clearly mark which components/features are MVP vs. later phases
- **Phase boundaries** — how the system grows from MVP to full-suite

1. **Draft the full design** — Write all sections into `DESIGN.md` using the Write tool
2. **Polish with writing skill** — Invoke the `writing` skill to improve clarity and conciseness of DESIGN.md before presenting
3. **Present for review** — Use `AskUserQuestion` with `preview` showing the complete design. Options: "Approve", "Needs changes"
4. **If "Needs changes"** — Ask what to change, revise `DESIGN.md`, then present again
5. **If "Approve"** — Proceed to Create TODO.md (step 6), then arch-review

The file is the source of truth — never present design content only in conversation text without also writing it to the file.

## Create TODO.md

After the user approves the design and before invoking arch-review, create `docs/<topic>/TODO.md` by extracting actionable items from DESIGN.md:

1. **Extract from scope & phases** — every feature, component, or capability listed in the design becomes a TODO item
2. **Mark MVP items** — items in the MVP phase get `[MVP]` prefix
3. **Mark deferred items** — items in later phases get their phase label (e.g., `[Phase 2]`, `[Phase 3]`)
4. **Include non-functional items** — testing setup, observability, documentation tasks from the design

Format:

```markdown
# TODO — <topic>

Generated from DESIGN.md. Updated by planning and implementation phases.

## MVP

- [ ] [MVP] <item from design>
- [ ] [MVP] <item from design>

## Phase 2

- [ ] [Phase 2] <item from design>

## Phase 3

- [ ] [Phase 3] <item from design>

## Non-functional

- [ ] <testing, observability, docs items>
```

This file is the living checklist. Planning refines it. Implementation marks items done.

## Auto-Invoke Arch-Review

After the user approves the design and TODO.md is created:

1. **Invoke `arch-review` skill** — it reviews DESIGN.md against architecture quality attributes
2. **Arch-review auto-updates DESIGN.md** — issues and decisions from the review are incorporated directly into the design document

The user does NOT need to manually trigger arch-review. It runs automatically as part of the brainstorming flow.

## Sci-Review Detection

After arch-review completes, scan the design for algorithmic/scientific content. Check if the design involves ANY of:

- Algorithm design or implementation (sorting, searching, graph algorithms, optimization)
- Numerical methods (floating-point arithmetic, interpolation, solvers, convergence)
- Scientific computing (physics simulation, geometry processing, mesh operations)
- AI/ML components (training loops, loss functions, data pipelines)
- Mathematical transformations (matrix operations, coordinate systems, projections)
- Performance-critical algorithms where complexity analysis matters

**If algorithmic/scientific content is detected:**

1. **Invoke `sci-review` skill** — it reviews the design for correctness, numerical stability, complexity, and edge cases
2. **Update DESIGN.md** — incorporate sci-review findings into the design
3. **Save SCI-REVIEW.md** alongside the design

The user does NOT need to manually trigger sci-review. It runs automatically when algorithmic content is detected.

**If NO algorithmic/scientific content** — skip sci-review and proceed to planning.

## Terminal State

After arch-review (and sci-review if applicable) completes and DESIGN.md is updated, invoke the `planning` skill. Do NOT invoke any other implementation skill. The ONLY skill you invoke after brainstorming is planning.

## Key Principles

- **Always use `AskUserQuestion`** — NEVER ask a question as plain text
- **One question at a time** — Don't overwhelm with multiple questions
- **Multiple choice preferred** — Use `options` in `AskUserQuestion` when possible
- **Use `preview` for comparisons** — When presenting approaches or design sections
- **Full-suite first, MVP second** — Design the whole system, then scope MVP within it
- **YAGNI ruthlessly within each phase** — But don't ignore future phases in the design
- **Explore alternatives** — Always propose 2-3 approaches before settling
- **Incremental validation** — Present design, get approval before moving on
