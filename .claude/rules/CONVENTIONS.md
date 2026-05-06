## Conventions

### Git

> This convention should be passed to all sub-agents.

- Use `git-commit` skill when you want to create a commit. (DO NOT use `Co-Authored-By` signature)
- Instead of running compound commands like `cd <path> && git <command>`, you should use the `-C` flag (e.g., `git -C <path> status`)
- DO NOT include `🤖 Generated with Claude Code` signature when creating PR.

### Markdown Documentation

- User `mermaid` to represent diagrams rather than ASCII.

### Implementation

- Use `subagent-driven-development` skill on implementation.
- Use `planning` skill before move to implementation.
- Use `brainstorming` skill if the question need further clarification, require a design or rich discussion.
