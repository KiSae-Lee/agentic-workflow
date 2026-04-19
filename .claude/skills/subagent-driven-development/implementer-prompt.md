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
    4. Commit your work
    5. Self-review (see below)
    6. Report back

    Work from: [directory]

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
