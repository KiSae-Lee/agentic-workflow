---
name: design
description: "Use after grill-me has produced common.md and atdd.md, or when the user already has a clear spec. Reads common.md and atdd.md, then produces full-suite design specs under spec/. Explores approaches, drafts spec documents, then auto-invokes arch-review to validate before transitioning to planning."
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

# Design — Ideas Into Full-Suite Specs

## Overview

Turn shared understanding (common.md) and acceptance criteria (atdd.md) into full-suite design specifications. This is NOT MVP-scoped — design the complete system. Phasing is planning's job.

**Announce at start**: "I'm using the design skill."

<HARD-GATE>
- Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it.
- The design MUST be full-suite oriented. Design the whole system.
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

If the user already has a clear spec, a detailed description, or says something like "I know what I want, just help me structure it" — skip reading common.md and go straight to proposing approaches. Ask one confirmation question to validate your understanding, then move on.

## Checklist

You MUST complete these steps in order:

1. **Read inputs** — load `spec/common.md` and `spec/atdd.md`
2. **Propose 2-3 approaches** — with trade-offs and your recommendation
3. **Draft spec documents** — write full-suite specs under `spec/`
4. **Polish** — invoke writing skill to improve clarity and conciseness of each spec file
5. **Present and revise** — present to user, incorporate feedback, iterate until approved
6. **Refine atdd.md** — update acceptance criteria if design revealed new features or edge cases
7. **Auto-invoke arch-review** — arch-review validates the spec documents
8. **Transition to planning** — invoke planning skill to create implementation plan

All spec artifacts are saved under `spec/` at the project root.

## The Process

<HARD-GATE>
EVERY question to the user MUST use the `AskUserQuestion` tool. Do NOT output a question as plain text and move on — you will not receive a response. Plain-text questions are invisible to the user in skill mode. The ONLY way to get user input is through `AskUserQuestion`.
</HARD-GATE>

**Loading shared understanding:**

- Read `spec/common.md` produced by grill-me
- Read `spec/atdd.md` — the acceptance criteria define what "done" looks like
- If common.md doesn't exist, fall back to exploring project context and asking clarifying questions (one at a time via `AskUserQuestion`)
- Internalize: problem statement, key decisions, constraints, success criteria, assumptions, acceptance criteria

**Exploring approaches:**

- Propose 2-3 different approaches using `AskUserQuestion` with `options` and `preview` fields
- Use `preview` to show concrete examples (architecture sketches, code snippets, data flow diagrams)
- Put your recommended option first with "(Recommended)" appended to its label
- Include trade-offs in each option's `description`

**Drafting spec documents:**

Write design specs as **separate files** under `spec/`. Each spec file covers one aspect of the design. Scale each file to its complexity — a short file for a simple aspect, a detailed file for a nuanced one.

1. **Draft the spec files** — Write each spec document using the Write tool (see Spec Directory Structure below)
2. **Polish with writing skill** — Invoke the `writing` skill to improve clarity and conciseness of each spec file before presenting
3. **Present for review** — Use `AskUserQuestion` with `preview` showing an overview of all spec files. Options: "Approve", "Needs changes"
4. **If "Needs changes"** — Ask what to change, revise the relevant spec file(s), then present again
5. **If "Approve"** — Proceed to Refine atdd.md, then arch-review

The files are the source of truth — never present design content only in conversation text without also writing it to a file.

## Spec Directory Structure

Read `./spec-templates.md` for the full template with few-shot examples for each file.

```
spec/
  # Always create
  glossary.md                       ← shared vocabulary, ubiquitous language
  constraints.md                    ← technology, regulatory, organizational constraints
  architecture-decisions.md         ← ADRs: decision, context, alternatives, consequences
  overview.md                       ← C4 context + container, solution strategy, tech stack
  non-functional-requirements.md    ← performance, security, scalability, availability targets
  use-cases.md                      ← use case descriptions per Feature from atdd.md
  flows.md                          ← sequence diagrams, state diagrams, event flows (mermaid)

  # Create when applicable
  data-model.md                     ← ER diagrams, schema design (when there's persistence)
  api-design.md                     ← endpoints, contracts, error codes (when there's an API)
  deployment.md                     ← infrastructure, containers, network topology (when ops alignment matters)
  ui/                               ← HTML mockups + behavior descriptions (when there's a frontend)
    <page-name>.html
    <page-name>.md
```

**Rules:**
- **Always create the 7 core files** — even if short, they establish shared understanding
- **Skip conditional files** only when truly not applicable (no database = no `data-model.md`)
- **Add custom files as needed** — e.g., `security-model.md`, `event-bus.md`, `migration-strategy.md`
- **Use mermaid for all diagrams** — sequence diagrams, ER diagrams, state machines, flow charts
- **Each file must be self-contained** — readable on its own without requiring other spec files for context
- **Reference atdd.md Features** — spec files should trace back to the Features defined in atdd.md
- **Cross-reference between specs** — link to related files (e.g., use-cases.md → flows.md for the sequence diagram)

### glossary.md — Shared Vocabulary

Defines the ubiquitous language for the project. Every team — frontend, backend, QA, product, ops — must use these terms consistently in code, specs, and conversation. Prevents "what do you mean by X?" misalignment in large teams. Table format: Term, Definition, Context. Include domain-specific terms, abbreviations, and terms that have multiple meanings across teams (e.g., "plan" in billing vs. project management).

### constraints.md — Hard Boundaries

Documents non-negotiable constraints that limit design choices. Three categories: technology constraints (mandated tech stack, infrastructure), regulatory constraints (GDPR, SOC 2, data residency), and organizational constraints (team size, deadlines, integration mandates). Each constraint includes the reason and its impact on design. This file prevents "why can't we just use X?" debates — the answer is here.

### architecture-decisions.md — The "Why" Record

Architecture Decision Records (ADRs). Each record captures: Status (proposed/accepted/deprecated), Context (what situation prompted the decision), Decision (what was chosen), Alternatives considered (with rejection reasons), and Consequences (positive and negative trade-offs). This file prevents teams from re-debating settled decisions and helps new members understand the rationale behind the architecture. Append new ADRs as decisions are made — never delete old ones.

### overview.md — Architecture Entry Point

The first file anyone reads. Contains: C4 Context diagram (system boundaries and external actors), C4 Container diagram (applications, data stores, message queues), tech stack table (technology, layer, rationale), solution strategy (key architectural approach), and cross-cutting concerns (auth strategy, error handling, observability, logging). This file answers "what does the system look like and how is it built?" at a glance.

### non-functional-requirements.md — Measurable Quality Targets

Explicit, measurable targets that prevent teams from making contradictory assumptions. Four categories: performance (p95 latency, page load, query time), scalability (concurrent users, data volume, tenant count), availability (uptime SLA, RTO, RPO), and security (encryption, compliance, vulnerability scanning). Every requirement has a metric, a target threshold, and a measurement method. "Fast" is not a requirement — "p95 < 200ms measured by k6 load test" is.

### use-cases.md — Actor Interactions

Detailed use case descriptions organized by Feature (matching atdd.md). Each use case includes: actors involved, preconditions, trigger, main flow (numbered steps), alternative flows (branching scenarios and error paths), postconditions (system state after completion), and traceability back to atdd.md acceptance criteria. This file bridges the gap between "what the user wants" (atdd.md) and "how the system behaves."

### flows.md — Visual Behavior

All dynamic behavior diagrams in one place. Three diagram types: sequence diagrams (component interactions for critical flows), state diagrams (lifecycle of stateful entities like orders, invitations, sessions), and event flow diagrams (async event propagation between services). All diagrams use mermaid. One diagram per critical flow — don't diagram trivial CRUD. This file replaces the narrower `sequence-diagrams.md` and is the visual companion to use-cases.md.

### data-model.md — Persistence Design (when applicable)

Create when the system has a database. Contains: ER diagram (mermaid erDiagram with entities, relationships, field types), access patterns table (query, tables involved, required indexes), and data strategy notes (multi-tenancy approach, partitioning, soft-delete policy). This file is the contract between application code and the database — backend developers write queries against it, DBAs optimize indexes from it.

### api-design.md — Interface Contracts (when applicable)

Create when the system exposes an API. For each endpoint: HTTP method and path, description, auth requirements, request schema (JSON example), response schema (JSON example), and error table (error code, HTTP status, condition). This file is the contract between frontend and backend teams — frontend builds against it, backend implements it. Version the API path (e.g., `/api/v1/`).

### deployment.md — Production Topology (when applicable)

Create when ops/SRE alignment matters. Contains: deployment topology diagram (mermaid — infrastructure, services, connections), environment configuration table (env vars, descriptions, examples), and scaling strategy (component, strategy, trigger). This file answers "how does this run in production?" for the ops team and prevents developers from designing things that can't be deployed.

### ui/ directory — Frontend Specs (when applicable)

Create when the system has a user interface. For each page or screen, two files: `<page-name>.html` (static HTML mockup showing layout and structure, semantic HTML with minimal inline styling) and `<page-name>.md` (behavior description: purpose, layout sections, interaction table with element/action/result, states — loading/empty/populated/error, and responsive breakpoints). This is the contract between design and frontend engineering.

## Refine atdd.md

After the user approves the spec and before invoking arch-review, review and update `spec/atdd.md`:

1. **Add discovered criteria** — design work often reveals features or edge cases the customer didn't articulate during grilling. Add them as new user stories under the appropriate Feature.
2. **Add new Features** — if the design introduced capabilities not in the original atdd.md (e.g., admin tools, monitoring dashboards), add them as new Feature sections.
3. **Do NOT remove or rewrite existing items** — atdd.md is the customer's voice. Only add, never subtract.
4. **Do NOT add implementation details** — keep items user-observable.

## Auto-Invoke Arch-Review

After the user approves the spec and atdd.md is refined:

1. **Invoke `arch-review` skill** — it reviews the spec documents (especially `overview.md` and `data-model.md`) against architecture quality attributes
2. **Arch-review updates spec files** — issues and decisions from the review are incorporated into the relevant spec documents

The user does NOT need to manually trigger arch-review. It runs automatically as part of the design flow.

## Sci-Review Detection

After arch-review completes, scan the spec for algorithmic/scientific content. Check if the design involves ANY of:

- Algorithm design or implementation (sorting, searching, graph algorithms, optimization)
- Numerical methods (floating-point arithmetic, interpolation, solvers, convergence)
- Scientific computing (physics simulation, geometry processing, mesh operations)
- AI/ML components (training loops, loss functions, data pipelines)
- Mathematical transformations (matrix operations, coordinate systems, projections)
- Performance-critical algorithms where complexity analysis matters

**If algorithmic/scientific content is detected:**

1. **Invoke `sci-review` skill** — it reviews the design for correctness, numerical stability, complexity, and edge cases
2. **Update spec files** — incorporate sci-review findings into the relevant spec documents
3. **Save sci-review.md** alongside the spec

The user does NOT need to manually trigger sci-review. It runs automatically when algorithmic content is detected.

**If NO algorithmic/scientific content** — skip sci-review and proceed to planning.

## Terminal State

After arch-review (and sci-review if applicable) completes and spec files are updated, invoke the `planning` skill. Do NOT invoke any other implementation skill. The ONLY skill you invoke after design is planning.

## Key Principles

- **Always use `AskUserQuestion`** — NEVER ask a question as plain text
- **One question at a time** — Don't overwhelm with multiple questions
- **Multiple choice preferred** — Use `options` in `AskUserQuestion` when possible
- **Use `preview` for comparisons** — When presenting approaches or design sections
- **Full-suite design** — Design the whole system, phasing is planning's job
- **YAGNI ruthlessly** — But don't ignore capabilities the full design needs
- **Explore alternatives** — Always propose 2-3 approaches before settling
- **Incremental validation** — Present design, get approval before moving on
- **Spec files trace to atdd.md** — every spec file should reference the Features it supports
