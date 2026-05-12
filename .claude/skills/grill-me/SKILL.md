---
name: grill-me
description: "Interview the user relentlessly about a plan, idea, or design until reaching shared understanding. Produces common.md (shared understanding) and atdd.md (E2E acceptance checklist from the customer's perspective). Use when user wants to stress-test a plan, get grilled on their design, explore an idea before designing, or mentions 'grill me'."
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
- Do NOT produce design specs — that is design's job.
- The ONLY outputs are common.md and atdd.md.
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

1. **Ask what the user wants to build** — start by asking the user to describe their idea. Do NOT read files, rules, or codebase before hearing from the user first. Offer two options: "Describe my idea" (user types their idea via Other) and "I already have a spec" (skip to design fast path).
2. **Explore project context** — check files, docs, recent commits. Read the Bootstrap rules above.
3. **Interview relentlessly** — ask questions one at a time using `AskUserQuestion`
   - If a question can be answered by exploring the codebase, explore the codebase instead
   - Prefer multiple choice (use `options`) when possible — user can always pick "Other"
   - For each question, provide your opinionated recommendation
   - Cover: purpose, constraints, success criteria, stakeholders, risks, trade-offs, boundaries
   - Resolve each branch of the decision tree before moving to the next
   - **Extract acceptance criteria as you go** — when the user describes what they want to achieve, capture it as an ATDD item. Phrase it from the customer/user perspective, not the developer's.
4. **Confirm understanding** — summarize using `AskUserQuestion` with `preview` showing both the common.md and atdd.md drafts. Options: "Approve", "Needs changes"
5. **Save common.md and atdd.md** — write both approved documents
6. **Transition** — invoke the `design` skill

## Saving Artifacts

Determine the path from context: `spec/`. If the topic is ambiguous, ask the user with `AskUserQuestion` before saving.

Save both files:
- `spec/common.md`
- `spec/atdd.md`

### common.md Structure

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
[Anything unresolved that design/design must address]

## Assumptions
[What we're assuming to be true — must be validated during design]
```

### atdd.md Structure

atdd.md is an **E2E acceptance checklist from the customer's perspective**. It defines what "done" looks like in user-observable terms. It is NOT a developer task list.

Organize by **Feature** (a specific capability that delivers user value). Each checklist item is a **User Story** written as an acceptance criterion — a user-facing behavior that can be verified end-to-end.

```markdown
# [Topic] — Acceptance Criteria

Acceptance test items extracted during shared-understanding interview.
Organized by Feature, each item is a User Story as acceptance criterion.
Updated by design (refinement) and checked off during implementation
when E2E verification passes.

## Feature: <feature name>
- [ ] <user story as acceptance criterion>
- [ ] <user story as acceptance criterion>

## Feature: <feature name>
- [ ] <user story as acceptance criterion>
- [ ] <user story as acceptance criterion>
```

**Example:**

```markdown
## Feature: Authentication
- [ ] User can sign up with email and password
- [ ] User can log in and land on the dashboard
- [ ] After 5 failed attempts, account locks with recovery instructions

## Feature: Order Management
- [ ] User can add items to cart and see updated total
- [ ] User can complete checkout with credit card
- [ ] User receives confirmation email within 1 minute
```

**Rules:**
- Group by **Feature** — a specific capability that delivers user value
- Each item is a **User Story** — written from the user/customer perspective, not the developer's
- Each item must be verifiable via an E2E test or manual acceptance check
- Do NOT include implementation details (file paths, function names, etc.)
- Do NOT group by delivery phase — phasing is planning's job, not the customer's
- Items are checked off during implementation when the acceptance test passes

## Terminal State

The terminal state is invoking `design`. Do NOT invoke planning, subagent-driven-development, or any other skill. The ONLY skill you invoke after grill-me is design.
