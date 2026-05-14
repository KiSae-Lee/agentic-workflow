## Conventions

### Package Manager

- Use `pnpm` for node.js.
- Use `uv` for Python.

### Git

> This convention should be passed to all sub-agents.

- Use `git-commit` skill when you want to create a commit. (DO NOT use `Co-Authored-By` signature)
- Instead of running compound commands like `cd <path> && git <command>`, you should use the `-C` flag (e.g., `git -C <path> status`)
- DO NOT include `🤖 Generated with Claude Code` signature when creating PR.

### Markdown Documentation

- User `mermaid` to represent diagrams rather than ASCII.

### Implementation

Skill invocation order is managed by the `control-tower` skill. Use `/control-tower` to enter the workflow.
