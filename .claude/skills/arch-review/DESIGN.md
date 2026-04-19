# Architect Review Skill — Design

## Overview

A project-agnostic, interactive skill that reviews design documents (DESIGN.md, architecture specs) against fundamental software architecture quality attributes before implementation begins. Modeled after the sci-review skill's interactive pattern: walks through issues one section at a time, one issue per `AskUserQuestion`, with opinionated recommendations.

**Name:** `arch-review`
**Trigger phrases:** "review the architecture", "architecture review", "review the design", "design review"
**Output:** `docs/<topic>/ARCH-REVIEW.md`

## What It Is / What It Is Not

| Is | Is Not |
|---|---|
| Pre-implementation design document review | Code review (that's `code-review`) |
| Architecture quality attribute evaluation | Algorithm/numerical correctness review (that's `sci-review`) |
| Interactive decision-making with the user | Autonomous agent that produces a report |
| Project-agnostic (works on any codebase) | Tied to a specific tech stack or framework |

## Skill Configuration

```yaml
name: arch-review
version: 1.0.0
description: |
  Software architecture design review. Evaluate design documents against
  architecture quality attributes organized in five categories: structural
  (integrity, correctness, completeness, simplicity), runtime (availability,
  security, usability, interoperability), development-time (buildability,
  testability, maintainability, reusability), evolutionary (modifiability,
  agility), and economic (TCO/ROI, standards). Interactive walk-through with
  opinionated recommendations. Use when asked to "review the architecture",
  "review the design", "architecture review", or "lock in the design".
allowed-tools:
  - Read
  - Write
  - Grep
  - Glob
  - AskUserQuestion
  - Bash
  - WebSearch
```

## Engineering Preferences

Mirrored from sci-review with architecture-specific additions:

- Conceptual integrity is non-negotiable — a fast inconsistent design is worse than a slow coherent one.
- Simplicity over cleverness. Every abstraction must earn its keep.
- Minimal surface area: achieve the goal with the fewest modules, interfaces, and dependencies.
- DRY at the design level — flag duplicated responsibilities aggressively.
- Explicit constraints over implicit assumptions.
- Buildability matters: a design that can't be built incrementally by a team is not a good design.
- YAGNI ruthlessly — defer decisions that don't need to be made now.

## Cognitive Patterns — How Strong Architects Think

Not a checklist. These are instincts to apply throughout the review.

1. **Conceptual integrity above all** — A system designed by one mind (or following one vision) is more usable than a patchwork of individually excellent ideas. (Brooks: "conceptual integrity is the most important consideration in system design.")
2. **Essential vs. accidental complexity** — Every design introduces complexity. Distinguish what's inherent to the problem (essential) from what's introduced by the solution (accidental). Minimize the latter relentlessly.
3. **Change-case thinking** — For every design decision, ask: "What happens when X changes?" If the answer is "rewrite three modules," the abstraction boundary is wrong.
4. **Dependency direction instinct** — Dependencies should point toward stability and abstraction, away from volatility and concreteness. If a stable module depends on a volatile one, invert it.
5. **Interface segregation** — No consumer should be forced to depend on methods it doesn't use. Fat interfaces signal mixed responsibilities.
6. **Blast radius awareness** — Every decision has a blast radius. A change in module A should not cascade to modules B, C, D. If it does, the coupling is too tight.
7. **Stakeholder empathy** — Different stakeholders (developers, operators, end users, business) have different concerns. A good architecture addresses all without sacrificing any.
8. **Build-vs-buy discipline** — Before designing a component, ask: does a well-tested solution already exist? Custom is justified only when domain-specific structure demands it.
9. **Incremental delivery bias** — Prefer designs that can be built, tested, and shipped in thin vertical slices over designs that require everything to be done before anything works.
10. **Standards awareness** — When a recognized standard or pattern exists for a problem, use it unless you can articulate a concrete reason not to. Standards reduce communication overhead and onboarding cost.

## Review Process

### Step 0: Scope & Feasibility Challenge

Before reviewing anything, answer:

1. **What is the system/feature being designed?** State it in one sentence. If you can't, the design is underspecified.
2. **Who are the stakeholders and what are their primary concerns?** (developers: buildability; operators: deployability; business: cost/time; users: functionality)
3. **What existing designs, patterns, or systems already solve this or a closely related problem?** Search if WebSearch is available.
4. **What existing code/modules already partially solve each sub-problem?** Can we extend rather than rebuild?
5. **Complexity check:** If the design introduces more than 5 new modules or defines more than 10 new interfaces, challenge whether the same goal can be achieved with fewer moving parts.
6. **Completeness check:** Are all stated requirements covered? Are non-functional requirements (performance, scalability, security) addressed or explicitly deferred?
7. **Input/Output contracts:** Are module boundaries, data formats, and error contracts precisely defined? Ambiguous contracts cause the most integration bugs.

If the complexity check triggers, proactively recommend scope reduction — explain what's overbuilt, propose a minimal version, ask whether to reduce or proceed.

**Critical: Once a scope recommendation is accepted or rejected, commit fully. Do not re-argue.**

### Section 1: Structural Quality Attributes

Evaluate the design's foundational coherence — the qualities that determine whether the architecture holds together as a unified whole.

#### Conceptual Integrity

- **Consistent abstractions:** Do all components use the same mental model? Or does the design mix paradigms (e.g., event-driven in one module, request-response in another) without justification?
- **Uniform naming and conventions:** Are concepts named consistently across the design? Does the same thing have different names in different places?
- **Single guiding vision:** Is there one coherent architectural style, or is it a patchwork of different approaches?
- **Pattern consistency:** If the design uses a pattern (e.g., ports-and-adapters, CQRS), is it applied uniformly or selectively without explanation?
- **Conceptual economy:** Are new concepts introduced only when necessary? Every new abstraction is a concept the team must learn and maintain.
- **Communication clarity:** Can the design be explained to a new team member in 10 minutes using a single diagram?

#### Correctness & Completeness

- **Requirement coverage:** Does every stated requirement map to at least one component/module? Are there orphaned requirements?
- **Constraint satisfaction:** Are hard constraints (performance, security, compliance) addressed in the design, not deferred to "implementation details"?
- **Failure mode coverage:** For each component, what happens when it fails? Is failure handling designed, not assumed?
- **Data integrity:** Are data consistency guarantees explicit? Is there a clear owner for each piece of state?
- **Contract completeness:** Are inter-module contracts (inputs, outputs, errors, invariants) fully specified? Missing error contracts are the #1 integration bug source.
- **Edge case awareness:** Are boundary conditions identified? (empty inputs, max scale, concurrent access, partial failures)
- **Assumption inventory:** What does the design assume about its environment? Are assumptions validated or silently trusted?

#### Complexity & Simplicity

- **Essential vs. accidental complexity:** Is each piece of complexity inherent to the problem or introduced by the solution? Flag accidental complexity.
- **Coupling analysis:** Map dependencies between modules. Are there circular dependencies? Are there modules with fan-out > 5 (depends on too many things)?
- **Cognitive load:** Can a developer understand one module without understanding all the others? High cognitive coupling = bad boundaries.
- **Abstraction fitness:** Are abstractions at the right level? Too abstract = hard to understand. Too concrete = hard to change.
- **YAGNI compliance:** Are there components, interfaces, or extension points that serve no current requirement? Flag them.
- **Indirection budget:** Every layer of indirection has a cost. Is each indirection justified by a concrete benefit (testability, replaceability, etc.)?
- **Configuration complexity:** How many knobs does the design expose? More configuration = more states to test and support.

**STOP.** One issue per `AskUserQuestion`. Present options, state recommendation, explain WHY. Proceed only after ALL issues resolved.

### Section 2: Runtime Quality Attributes

Evaluate the qualities that manifest when the system is running in production.

#### Availability

- **Uptime expectations:** What is the expected availability? Are there single points of failure?
- **Redundancy & failover:** Does the design include redundancy, failover, or graceful degradation strategies?
- **Partial failure handling:** What happens during partial outages? Can the system continue in a degraded mode?

#### Security

- **Trust boundaries:** Are trust boundaries identified and drawn in the design?
- **Input validation:** Is input validated at system boundaries — not deep inside business logic?
- **Authentication & authorization:** Are authn/authz concerns addressed in the design, not deferred to "implementation"?
- **Data protection:** Are secrets management, data-at-rest/in-transit encryption, and least-privilege access designed in — not bolted on?

#### Usability

- **User experience consideration:** Does the design consider the end-user experience at the architecture level?
- **Error states & recovery:** Are error states, feedback loops, and recovery paths designed?
- **Accessibility:** Are accessibility constraints addressed or explicitly deferred?

#### Interoperability

- **External integration:** Does the system need to exchange data with external systems? Are integration points identified?
- **Protocols & contracts:** Are integration protocols, data formats, and API contracts defined?
- **Standards usage:** Are recognized standards (REST, gRPC, OpenAPI, etc.) used where applicable?

**STOP.** Interactive resolution before proceeding.

### Section 3: Development-time Quality Attributes

Evaluate the qualities that affect the team's ability to build, test, change, and maintain the system.

#### Buildability & Modularity

- **Module boundaries:** Can each module be developed, tested, and deployed independently? Or are there implicit coupling points?
- **Parallel workstreams:** Can multiple developers/teams work on different modules simultaneously without blocking each other?
- **Interface contracts:** Are module interfaces defined before implementation? Are they minimal (only what consumers need)?
- **Dependency management:** Are third-party dependencies justified? Are they stable? What happens if one is abandoned?
- **Incremental delivery:** Can the design be built in thin vertical slices? Is there a meaningful "walking skeleton" that can be delivered first?
- **Build system impact:** Does the design require build system changes, new tooling, or special compilation steps?

#### Testability

- **Isolation:** Can each component be tested in isolation? Are test seams (ports, interfaces, dependency injection points) designed in?
- **Multi-level testing:** Can the system be tested at unit, integration, and system levels without requiring the full stack?
- **Determinism:** Are test-hostile patterns (global state, time-dependence, non-deterministic ordering) avoided or isolated?

#### Maintainability

- **Understandability:** Is the design understandable to a new team member? Are responsibilities clear and localized?
- **Debuggability:** Can operators trace problems to their source? Is the debugging experience considered in the design?
- **Observability hooks:** Are logging, metrics, and tracing designed into the architecture — not afterthoughts?

#### Reusability

- **Reusable components:** Are there components designed to be reused across modules or projects?
- **Assumption-free:** Are reusable components free of project-specific assumptions?
- **Over-generalization check:** Conversely, is there unnecessary generalization — building for reuse that nobody will use?

**STOP.** Interactive resolution before proceeding.

### Section 4: Evolutionary Quality Attributes

Evaluate the design's ability to accommodate change over time.

#### Modifiability

- **Change accommodation:** Can the design accommodate likely changes without cascading modifications?
- **Isolation of volatility:** Are change-prone areas isolated behind stable interfaces?
- **Proportional cost:** Is the modification cost proportional to the change size?

#### Agility & Evolvability

- **Change impact radius:** For the 3 most likely future changes, how many modules need modification? If > 2, the boundary is wrong.
- **Extension points:** Where does the design expect growth? Are those points actually extensible without modification (open-closed)?
- **Backward compatibility:** If this replaces or extends existing functionality, is there a migration path? Can old and new coexist?
- **Versioning strategy:** How are breaking changes to interfaces handled? Is there a versioning or deprecation strategy?
- **Technology lock-in:** Is the design coupled to a specific technology, or can the technology be swapped at the boundary?
- **Reversibility:** Which design decisions are easily reversible and which are one-way doors? Are one-way doors identified and justified?

**STOP.** Interactive resolution before proceeding.

### Section 5: Cost/Value Alignment

Evaluate the economic and organizational fitness of the design.

- **TCO awareness:** What is the ongoing maintenance cost of this design? Are there components that will require disproportionate maintenance?
- **ROI of complexity:** For each complex component, is the value it provides proportional to its cost?
- **Build vs. buy:** Are there components being designed from scratch that could be replaced by existing libraries/services?
- **Standards compliance:** Does the design follow recognized patterns and standards? Deviating from standards increases onboarding cost and bug surface.
- **Team capability fit:** Can the current team build and maintain this design? Does it require expertise the team doesn't have?
- **Operational cost:** What does this design cost to run, monitor, and debug in production?

**STOP.** Interactive resolution before proceeding.

### Quality Attribute Summary Matrix

After all sections, produce a summary matrix:

```
Category        │ Quality Attribute      │ Addressed │ Deferred │ N/A │ Notes
────────────────┼────────────────────────┼───────────┼──────────┼─────┼──────────
Structural      │ Conceptual Integrity   │           │          │     │
                │ Correctness            │           │          │     │
                │ Completeness           │           │          │     │
                │ Simplicity             │           │          │     │
────────────────┼────────────────────────┼───────────┼──────────┼─────┼──────────
Runtime         │ Availability           │           │          │     │
                │ Security               │           │          │     │
                │ Usability              │           │          │     │
                │ Interoperability       │           │          │     │
────────────────┼────────────────────────┼───────────┼──────────┼─────┼──────────
Development     │ Buildability           │           │          │     │
                │ Testability            │           │          │     │
                │ Maintainability        │           │          │     │
                │ Reusability            │           │          │     │
────────────────┼────────────────────────┼───────────┼──────────┼─────┼──────────
Evolutionary    │ Modifiability          │           │          │     │
                │ Agility                │           │          │     │
────────────────┼────────────────────────┼───────────┼──────────┼─────┼──────────
Economic        │ TCO/ROI                │           │          │     │
                │ Standards Compliance   │           │          │     │
```

For each "Deferred" attribute, require a one-line justification. Deferring without justification = **flag**.

## Interaction Rules

Mirrored from sci-review:

- **One issue = one `AskUserQuestion` call.** Never combine multiple issues.
- Describe the problem concretely, with references to the design document where applicable.
- Present 2-3 options, including "do nothing" where reasonable.
- For each option: effort, risk, and impact in one line.
- Map reasoning to the engineering preferences or cognitive patterns above.
- Label with issue NUMBER + option LETTER (e.g., "3A", "3B").
- **Escape hatch:** If a section has no issues, say so and move on.

## Required Outputs

### "NOT in scope" section
Work considered and explicitly deferred, with one-line rationale each.

### "What already exists" section
Existing code, libraries, patterns, or systems that already solve sub-problems. Flag unnecessary rebuilds.

### "Key assumptions" section
Every assumption the design makes about inputs, environment, or system behavior. For each: validated at runtime or silently trusted.

### "Failure modes" section
For each architectural component:
1. One realistic failure scenario
2. Is failure handling designed for it?
3. Would the user/operator see a clear signal or get silent degradation?

Silent unhandled failures = **critical gap**.

### "Decision log" section
Each decision made during the review: what was decided, which option was chosen, and why. This is the primary artifact for future reference.

### Completion summary

```
- Step 0: Scope Challenge — ___ (accepted as-is / reduced)
- Structural Qualities: ___ issues found
- Runtime Qualities: ___ issues found
- Development-time Qualities: ___ issues found
- Evolutionary Qualities: ___ issues found
- Cost/Value Alignment: ___ issues found
- Quality Attributes: ___ addressed / ___ deferred / ___ N/A
- NOT in scope: written
- What already exists: written
- Key assumptions: ___ validated / ___ unvalidated
- Failure modes: ___ critical gaps flagged
- Decision log: ___ decisions recorded
```

### Unresolved decisions
If the user does not respond or interrupts, note which decisions were left unresolved as "Unresolved decisions that may bite you later." Never silently default.

## Saving the Review

After full review completion, save to:

```
docs/<topic>/ARCH-REVIEW.md
```

Where `<topic>` is a short kebab-case name derived from the design being reviewed. If ambiguous, ask with `AskUserQuestion`.

The saved file is a clean, self-contained document — no conversation artifacts.

## Diagrams

The review should use ASCII diagrams for:
- Module dependency graphs (as identified during complexity review)
- Data flow paths (as identified during correctness review)
- Change impact maps (as identified during agility review)

Diagram maintenance is part of the change — flag stale diagrams in the design document.

## Relationship to Other Skills

```
brainstorming → planning → [sci-review] → subagent-driven-development → [arch-review] → code-review
                              ↑                                             ↑
                        algorithms/numerics                          architecture/design
                        (before implementation)                      (before implementation)
```

- **sci-review**: Evaluates algorithmic correctness, numerical stability, convergence. Use for computational/scientific concerns.
- **arch-review**: Evaluates architecture quality attributes. Use for structural/design concerns.
- **code-review**: Evaluates implemented code quality. Use after code is written.

These three can be composed: a design with algorithmic components might get both `sci-review` (for the algorithm) and `arch-review` (for the architecture).
