# Implementer Subagent Prompt Template

Use this template when dispatching an implementer subagent.

```
Task tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N: [task name]

    ## Task Description

    [FULL TEXT of task from plan - paste it here, don't make subagent read file]

    ## Context

    [Scene-setting: where this fits, dependencies, architectural context]

    ## Before You Begin

    If you have questions about:
    - The requirements or acceptance criteria
    - The approach or implementation strategy
    - Dependencies or assumptions
    - Anything unclear in the task description

    **Ask them now.** Raise any concerns before starting work.

    ## Your Job

    Once you're clear on requirements:
    1. Implement exactly what the task specifies
    2. Write tests (following TDD if task says to)
    3. Verify implementation works
    4. Register any tech debt you incur or discover (see Tech Debt section below)
    5. Commit your work
    6. Self-review (see below)
    7. Report back

    Work from: [directory]

    ## Conventions

    Follow `.claude/skills/control-tower/rules/CONVENTIONS.md`.

    ## Tech Debt Context (injected by dispatcher)

    **Assigned ID range:** [e.g. `TD-040` through `TD-049` — use ONLY these IDs for new debt in this task]
    **Adjacent open debt (Boy Scout candidates):** [e.g. `TD-017` at `src/payment/gateway.go:88`; or "none"]
    **Approver for deliberate debt:** [defaults to the human user — never yourself]

    ## Tech Debt Convention

    **The schema, lifecycle, and update rules are defined in `.claude/skills/subagent-driven-development/tech-debt-template.md`.** Read it before reporting debt. Summary of what you MUST do:

    ### Registering new debt

    Trigger: you take a shortcut in code (add `TODO`/`FIXME`/`HACK`), OR you discover a non-code gap (design coupling, missing doc, brittle infra setup) that someone will pay interest on later.

    1. **Pick the next ID** from your assigned range only (see "Tech Debt Context" above). Start at the lowest; if you register multiple debts, increment within the range. Do NOT pick IDs outside the assigned range — that causes collisions with parallel implementers.
    2. **If category is `code` — annotate code in the SOP-A2-01 format.** Bare TODOs without an ID will fail code review:
       ```
       // TODO(#TD-042): swap polling for webhook once provider supports it
       // FIXME(#TD-017): off-by-one on empty-list edge case
       // HACK(#TD-029): hardcoded retry count, lift to config in payment-v2
       ```
       **If category is `design` / `doc` / `infra`** — no code annotation is required; the `Location` field in the entry names the component, doc, or system instead of a file:line. Code review will not flag absence of annotation for these categories.
    3. **Do NOT modify `spec/tech-debt.md` directly.** Parallel worktrees would conflict on that file. Instead, include the full entry text in your report (see Report Format below). The dispatcher writes `spec/tech-debt.md` at phase finalization, after all worktrees have merged.
    4. **Deliberate debt extra requirement.** If the Fowler quadrant is `reckless-deliberate` or `prudent-deliberate`, you are taking debt *on purpose*. Make sure your entry's `Reason` field is concrete (not "no time") and `Repayment target` is a real date or milestone. Report `Deliberate debt taken: yes` so the dispatcher includes a `## Deliberate Debt` section in `implementation-report.md` per SOP-A2-07 (the dispatcher will list the approver — never use your own name).

    ### Boy Scout rule

    "Adjacent open debt" above lists open TD entries near files you're touching. While completing your task, **try to close ≥1 adjacent debt** if it fits within ~30 minutes of extra work. Closing means: fix the code so the debt no longer applies, remove any `// TODO(#TD-NNN)` annotation (skip this step if it's a `design`/`doc`/`infra` entry with no code annotation), and report it in the "Closed" section of your report with a 1-sentence Outcome. The dispatcher moves the entry from Active to Closed in `spec/tech-debt.md`.

    Do not exceed 30 minutes on Boy Scout cleanup — if it's bigger, leave it for a dedicated repayment phase.

    ### Forbidden

    - Bare `TODO`, `FIXME`, `HACK` comments without `(#TD-NNN)` — code review will block.
    - Inventing fake IDs (outside your assigned range or referencing TD-NNN that doesn't exist after merge) — code review will block.
    - Directly editing `spec/tech-debt.md` — write nothing to that file; it's the dispatcher's responsibility.
    - Self-approval on deliberate debt — the approver must be someone other than you (defaults to the human user).

    **While you work:** If you encounter something unexpected or unclear, **ask questions**.
    It's always OK to pause and clarify. Don't guess or make assumptions.

    ## Before Reporting Back: Self-Review

    Review your work with fresh eyes. Ask yourself:

    **Completeness:**
    - Did I fully implement everything in the spec?
    - Did I miss any requirements?
    - Are there edge cases I didn't handle?

    **Quality:**
    - Is this my best work?
    - Are names clear and accurate (match what things do, not how they work)?
    - Is the code clean and maintainable?

    **Discipline:**
    - Did I avoid overbuilding (YAGNI)?
    - Did I only build what was requested?
    - Did I follow existing patterns in the codebase?

    **Testing:**
    - Do tests actually verify behavior (not just mock behavior)?
    - Did I follow TDD if required?
    - Are tests comprehensive?

    If you find issues during self-review, fix them now before reporting.

    ## Report Format

    When done, report:
    - What you implemented
    - What you tested and test results
    - Files changed
    - **Tech debt summary** (the dispatcher uses this to update `spec/tech-debt.md`):
      - **Opened** — for each TD-NNN you registered, give the full entry in this exact format:
        ```
        TD-NNN — <one-line summary>
        Category: code | design | doc | infra
        Fowler quadrant: reckless-deliberate | reckless-inadvertent | prudent-deliberate | prudent-inadvertent
        Principal: N MD
        Interest rate: N% (<observed or estimated basis>)
        Blast radius: High | Medium | Low
        Priority score: (Interest × RiskWeight) ÷ Principal = N.N
        Priority label: P0 | P1 | P2 | P3
        Location: path:line (or component name)
        Owner: self
        Repayment target: YYYY-MM-DD | milestone | unscheduled
        Reason: <concrete reason>
        Remediation sketch: <1-3 sentences>
        ```
      - **Closed** — for each TD-NNN you closed via Boy Scout rule:
        ```
        TD-NNN — <original summary>
        Commit: <hash>
        Outcome: <1-2 sentences on what changed>
        ```
      - **Deliberate debt taken?** yes / no (if yes: list TD-NNN; confirm Reason and Repayment target are concrete)
    - Self-review findings (if any)
    - Any issues or concerns

    ## TDD Methodology (Bundled Reference)

    ### The Iron Law

    ```
    NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
    ```

    Write code before the test? Delete it. Start over. No exceptions.

    ### Red-Green-Refactor Cycle

    1. **RED** — Write one minimal failing test showing what should happen
       - One behavior per test
       - Clear descriptive name
       - Real code, no mocks unless unavoidable
    2. **VERIFY RED** — Run test, confirm it fails for the expected reason (missing feature, not typo)
    3. **GREEN** — Write the simplest code to make the test pass. No extra features.
    4. **VERIFY GREEN** — Run test, confirm it passes. Confirm all other tests still pass.
    5. **REFACTOR** — Clean up while keeping tests green. Don't add behavior.
    6. **REPEAT** — Next failing test for next behavior.

    ### Testing Anti-Patterns to Avoid

    1. **Never test mock behavior** — Assert on real component behavior, not that a mock exists
    2. **Never add test-only methods to production classes** — Put test cleanup in test utilities
    3. **Never mock without understanding dependencies** — Know what side effects the real method has before mocking
    4. **Never use incomplete mocks** — Mirror the real API structure completely, not just fields you think you need
    5. **Never treat tests as afterthought** — Testing is part of implementation, not optional follow-up

    ### Gate Questions Before Mocking

    - "Am I testing real behavior or mock existence?"
    - "Is this method only used by tests?" (if yes, don't add it to production)
    - "What side effects does the real method have? Does my test depend on them?"
    - "Does my mock match the complete real API structure?"

    ### Red Flags — STOP and Start Over

    - Code written before test
    - Test passes immediately (you're testing existing behavior)
    - Can't explain why test failed
    - Mock setup is >50% of test
    - Rationalizing "just this once"
```
