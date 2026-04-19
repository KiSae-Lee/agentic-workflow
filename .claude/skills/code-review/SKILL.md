---
name: code-review
description: Use when reviewing code changes or codebase quality for production readiness
---

# Code Review

Dispatch a code-reviewer subagent to review code changes across five dimensions.

**Announce at start**: "I'm using the code-review skill."

## Process

1. Identify the review scope (commit, branch, or files to review)
2. Dispatch a **code-reviewer subagent** with the full review methodology:
   - Use Agent tool (general-purpose type)
   - Description: "Code review: [scope description]"
   - Prompt: Read and include the full contents of `./code-review-agent-prompt.md`
   - Provide the subagent with: review scope, commit/branch info, file list
3. The subagent performs: DISCOVER -> READ -> ANALYZE -> VERIFY -> REPORT
4. The subagent saves the report to `docs/<commit-identifier>-YY-MM-DD-HH:MM:SS-code-review.md`
5. Report the assessment summary back to the user (verdict + issue counts only)

## When to Use

- Reviewing recent changes (git diff against base branch)
- Reviewing a feature branch before merge
- Auditing codebase quality
- After refactoring or completing major features

## Key Rule

**Never run the review in the main context.** Always dispatch a subagent. The review methodology is heavy (~500 lines) and self-contained — it doesn't need user interaction during execution.
