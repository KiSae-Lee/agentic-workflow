---
name: brainstorming
description: "Use this before multi-step features, ambiguous requirements, or when the user needs help clarifying what to build. Explores user intent, requirements, and design before implementation. Trigger when the user describes a feature idea without a clear spec, asks 'how should I build X', or starts a task that involves multiple components or trade-offs. Do NOT trigger for simple, well-defined changes like renaming, adding a log line, or fixing a typo."
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

# Brainstorming Ideas Into Designs

## Overview

Help turn ideas into fully formed designs and specs through natural collaborative dialogue.

Start by understanding the current project context, then ask questions one at a time to refine the idea. Once you understand what you're building, present the design and get user approval.

<HARD-GATE>
- Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it.
</HARD-GATE>

## Fast Path

If the user already has a clear spec, a detailed description, or says something like "I know what I want, just help me structure it" — skip the exploration phase (steps 1-2) and go straight to proposing approaches or drafting the design. Ask one confirmation question to validate your understanding, then move on. Not every idea needs deep excavation.

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Explore project context** — check files, docs, recent commits
2. **Ask clarifying questions** — one at a time, understand purpose/constraints/success criteria
3. **Propose 2-3 approaches** — with trade-offs and your recommendation
4. **Draft design** — write a complete `DESIGN.md`, then present it to the user for holistic review
5. **Revise** — incorporate user feedback, iterate until approved
6. **Transition to implementation planning** — invoke planning skill to create implementation plan

Determine the design file path from context: use `<relevant-module-path>/docs/<topic>/DESIGN.md`. If the module path is ambiguous, ask the user with `AskUserQuestion` before proceeding.

**The terminal state is invoking planning.** Do NOT invoke frontend-design, mcp-builder, or any other implementation skill. The ONLY skill you invoke after brainstorming is planning.

## The Process

<HARD-GATE>
EVERY question to the user MUST use the `AskUserQuestion` tool. Do NOT output a question as plain text and move on — you will not receive a response. Plain-text questions are invisible to the user in skill mode. The ONLY way to get user input is through `AskUserQuestion`.
</HARD-GATE>

**Understanding the idea:**

- Check out the current project state first (files, docs, recent commits)
- Ask clarifying questions one at a time using `AskUserQuestion`
- Prefer multiple choice (use `options` in `AskUserQuestion`) when possible — the user can always pick "Other" for a custom answer
- Only one question per `AskUserQuestion` call — if a topic needs more exploration, break it into multiple rounds
- Focus on understanding: purpose, constraints, success criteria
- WAIT for the user's answer before proceeding to the next question

**Exploring approaches:**

- Propose 2-3 different approaches using `AskUserQuestion` with `options` and `preview` fields
- Use `preview` to show concrete examples (architecture sketches, code snippets, data flow diagrams)
- Put your recommended option first with "(Recommended)" appended to its label
- Include trade-offs in each option's `description`

**Presenting the design:**

Write the complete design into the `DESIGN.md` file, covering relevant sections (architecture, components, data flow, error handling, testing — skip what doesn't apply). Scale each section to its complexity: a few sentences if straightforward, up to 200-300 words if nuanced.

1. **Draft the full design** — Write all sections into `DESIGN.md` using the Write tool
2. **Present for review** — Use `AskUserQuestion` with `preview` showing the complete design. Options: "Approve", "Needs changes"
3. **If "Needs changes"** — Ask what to change, revise `DESIGN.md`, then present again
4. **If "Approve"** — Invoke the `planning` skill to create the implementation plan

The file is the source of truth — never present design content only in conversation text without also writing it to the file.

## Key Principles

- **Always use `AskUserQuestion`** - NEVER ask a question as plain text. Every question must go through the tool
- **One question at a time** - Don't overwhelm with multiple questions
- **Multiple choice preferred** - Use `options` in `AskUserQuestion` when possible
- **Use `preview` for comparisons** - When presenting approaches or design sections, use the `preview` field to show concrete artifacts (ASCII diagrams, code snippets, config examples)
- **YAGNI ruthlessly** - Remove unnecessary features from all designs
- **Explore alternatives** - Always propose 2-3 approaches before settling
- **Incremental validation** - Present design, get approval before moving on
- **Be flexible** - Go back and clarify when something doesn't make sense
