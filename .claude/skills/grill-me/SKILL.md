---
name: grill-me
description: "Interview the user relentlessly about a plan, idea, or design until reaching shared understanding. Produces COMMON.md — the shared-understanding document that brainstorming consumes. Use when user wants to stress-test a plan, get grilled on their design, explore an idea before designing, or mentions 'grill me'."
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

# Grill Me — Build Shared Understanding

**Announce at start**: "I'm using the grill-me skill."

Interview the user relentlessly about every aspect of this idea/plan/design until we reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

<HARD-GATE>
- Do NOT invoke any implementation skill, write any code, scaffold any project, or take any design action.
- Do NOT produce DESIGN.md — that is brainstorming's job.
- The ONLY output is COMMON.md.
- EVERY question to the user MUST use the `AskUserQuestion` tool. Plain-text questions are invisible to the user in skill mode.
</HARD-GATE>

## Bootstrap: Read Rules

After learning what the user wants to build, read these rules to inform your questions:

| Rule file | Why |
|---|---|
| `.claude/skills/control-tower/rules/AWARENESS.md` | Know cross-service pitfalls and integration lessons to ask about |
| `.claude/skills/control-tower/rules/KEEP_IN_MIND.md` | Interface design principles and testability to probe |
| `.claude/skills/control-tower/rules/ARCHITECTURE.md` | Architectural patterns to ground the conversation |

## The Process

1. **Ask what the user wants to build** — start by asking the user to describe their idea. Do NOT read files, rules, or codebase before hearing from the user first. Offer two options: "Describe my idea" (user types their idea via Other) and "I already have a spec" (skip to brainstorming fast path).
2. **Explore project context** — check files, docs, recent commits. Read the Bootstrap rules above.
3. **Interview relentlessly** — ask questions one at a time using `AskUserQuestion`
   - If a question can be answered by exploring the codebase, explore the codebase instead
   - Prefer multiple choice (use `options`) when possible — user can always pick "Other"
   - For each question, provide your opinionated recommendation
   - Cover: purpose, constraints, success criteria, stakeholders, risks, trade-offs, boundaries
   - Resolve each branch of the decision tree before moving to the next
4. **Confirm understanding** — summarize the shared understanding using `AskUserQuestion` with `preview` showing the full COMMON.md draft. Options: "Approve", "Needs changes"
5. **Save COMMON.md** — write the approved document
6. **Transition** — invoke the `brainstorming` skill

## Saving COMMON.md

Determine the path from context: `docs/<topic>/COMMON.md`

If the topic is ambiguous, ask the user with `AskUserQuestion` before saving.

### COMMON.md Structure

```markdown
# [Topic] — Shared Understanding

## Problem Statement
[What problem are we solving and why]

## Key Decisions
[Each decision made during the interview, with chosen option and rationale]

## Constraints & Boundaries
[Hard constraints, out-of-scope items, non-negotiable requirements]

## Success Criteria
[How we know this is done and done well]

## Open Questions
[Anything unresolved that brainstorming/design must address]

## Assumptions
[What we're assuming to be true — must be validated during design]
```

## Terminal State

The terminal state is invoking `brainstorming`. Do NOT invoke planning, subagent-driven-development, or any other skill. The ONLY skill you invoke after grill-me is brainstorming.
