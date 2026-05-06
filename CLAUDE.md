# Title Here

## Overview

Your project description here.

## Principles

Follow the guidelines defined in `rules/`:

- **`.claude/rules/ARCHITECTURE.md`** — Clean Architecture best practices: dependency rule, layer responsibilities, DIP with traits, DTO isolation via `From`/`Into`, and error translation across boundaries.
- **`.claude/rules/DOCUMENTATION.md`** — Rust documentation standards: action-oriented summaries, thorough parameter docs, enum variant docs, `# Errors`/`# Panics` sections, and doc tests.
- **`.claude/rules/WORKFLOW.md`** — Development workflow: explore codebase, brainstorm design, plan tasks, optional sci-review, implement via subagent-driven-development, and code-review before finalizing.
- **`.claude/rules/OBSERVABILITY.md`** — Logging & observability: structured logging with `tracing`, strict log level discipline, correlation IDs, no PII in logs, and actionable error messages.
- **`.claude/rules/CONVENTIONS.md`** — Project conventions: git commit rules, naming standards, and cross-platform guidelines.
- **`.claude/rules/AWARENESS.md`** — Lessons learned: cross-service contract verification, DB schema/ORM alignment, dependency pinning, proto generation, and startup readiness.

## Project Specific Rules

Follow `RULES.md`.

### Architecture

Follow `docs/MSA-DESIGN.md`.
