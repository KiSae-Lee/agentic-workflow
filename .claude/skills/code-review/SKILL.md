---
name: code-review
description: Use when reviewing code changes or codebase quality for production readiness
---

# Code Review

Dispatch a code-reviewer subagent to review code changes across five dimensions.

**Announce at start**: "I'm using the code-review skill."

## Bootstrap: Read Rules

Before dispatching the review subagent, read these rules and include relevant sections in the subagent prompt:

| Rule file | Why |
|---|---|
| `.claude/skills/control-tower/rules/ARCHITECTURE.md` | Architecture principles the reviewer checks against |
| `.claude/skills/control-tower/rules/CONVENTIONS.md` | Naming, git, style conventions to verify |
| `.claude/skills/control-tower/rules/DOCUMENTATION.md` | Doc standards to check in reviewed code |
| `.claude/skills/control-tower/rules/OBSERVABILITY.md` | Logging and tracing standards for production readiness |
| `.claude/skills/control-tower/rules/KEEP_IN_MIND.md` | Interface design, test quality, mocking boundaries |

**Procedure:** Read all listed rule files. Pass relevant excerpts to the code-review subagent as context.

## Process

1. Identify the review scope (commit, branch, or files to review)
2. Dispatch a **code-reviewer subagent** with the full review methodology:
   - Use Agent tool (general-purpose type)
   - Description: "Code review: [scope description]"
   - Prompt: Read and include the full contents of `./code-review-agent-prompt.md`
   - Provide the subagent with: review scope, commit/branch info, file list
3. The subagent performs: DISCOVER -> READ -> ANALYZE -> VERIFY -> REPORT
4. The subagent saves the report to `docs/<topic>/CODE-REVIEW.md`
5. Report the assessment summary back to the user (verdict + issue counts only)

## When to Use

- Reviewing recent changes (git diff against base branch)
- Reviewing a feature branch before merge
- Auditing codebase quality
- After refactoring or completing major features

## Key Rule

**Never run the review in the main context.** Always dispatch a subagent. The review methodology is heavy (~500 lines) and self-contained — it doesn't need user interaction during execution.
