## Workflow

### Project Design Workflow

```mermaid
flowchart TD
    A[PROJECT Idea] --> B{Does PROJECT_DESIGN.md exist?}
    B -- No --> C[Start brainstorm based on PROJECT/Revision idea]
    B -- Yes --> D[Read PROJECT_DESIGN.md]
    D --> C
    C --> E[Create/Update PROJECT_DESIGN.md]
    E --> F{Architecture review needed?}
    F -- Yes --> G[Start arch-review]
    G --> H[Create ARCH-REVIEW.md]
    H -- Update --> E
    F -- No --> I{Is related to scientific stuff?}
    I -- Yes --> J[Start sci-review]
    J --> K[Create SCI-REVIEW.md]
    K -- Update --> E
    I -- No --> L[Create/Update TODO.md]
    L --> M{Revision?}
    M -- "Yes: Ask for revision idea" --> C
    M -- No --> N[End of session]
```

<caution>
- Ask if you are confused the idea is belong to the `Project Design` or `Implementation`.
- `PROJECT_DESIGN.md` should include full-suite development plan and MVP development plan.
- `TODO.md` should clearify which item is belong to the MVP.
</caution>

### Implementation Workflow

```mermaid
flowchart TD
    A[Implementation Idea] --> B{Does CODEBASE_MAP.md exist?}

    subgraph Explore
        B -- Yes --> C[Read CODEBASE_MAP.md and skip exploring]
        B -- No --> D["Create CODEBASE_MAP.md (Using graph-generator skill --map)"]
        D --> C
    end

    subgraph Create specification
        C --> E[Use brainstorming skill]
        E --> F[DESIGN.md]
        F --> G{Revision for the design?}
        G -- No --> H[Use arch-review skill]
        H --> I[ARCH-REVIEW.md]
        I --> J[Use planning skill]
        J --> K["PLAN.md, TASK.md, NEXT_STEP.md"]
        K --> L{Revision for the plan?}
    end

    subgraph Sci-Review
        L -- No --> M{Algorithmic/scientific content?}
        M -- Yes --> N[Use sci-review skill]
        N --> O[SCI-REVIEW.md]
        O --> P[Use subagent-driven-development skill]
    end

    subgraph Implementation
        M -- No --> P
        P --> Q{Review?}
        Q -- Yes --> R["Use code-review skill (subagent)"]
        Q -- No --> S[IMPLEMENTATION_REPORT.md]
        R --> T[CODE-REVIEW.md]
        T --> U{Any Critical or Important issue?}
        U -- Yes --> V{Continue?}
        V -- Yes --> J
        V -- No --> S
        U -- No --> S
        S --> W[Update CODEBASE_MAP.md]
    end

    G -- Yes --> X[End of session]
    L -- Yes --> X
    W --> Y["Update pipelines/ (Using graph-generator skill)"]
    Y --> Z[Update TODO.md]
    Z --> X
```

<caution>
- Remove the worktree if it is created for parallel tasks after the implmentation.
</caution>

### Worktree Cleanup Workflow

After each subagent completes work in a worktree, clean up immediately. After all implementation is done, run a final sweep.

```mermaid
flowchart TD
    A[Subagent task completed] --> B[Merge worktree branch into main]
    B --> C["Remove worktree (git worktree remove)"]
    C --> D["Delete branch (git branch -d)"]
    D --> E[Next task or final sweep]
    E --> F{All tasks done?}
    F -- No --> G[Continue implementation]
    F -- Yes --> H["Final sweep: git worktree list"]
    H --> I{Any leftover worktrees?}
    I -- Yes --> J["Remove each with git worktree remove"]
    J --> K["Remove .claude/worktrees/ directory"]
    I -- No --> K
    K --> L[Clean]
```

**Commands reference:**

```bash
# List all worktrees
git worktree list

# Remove a specific worktree (after merging its branch)
git worktree remove .claude/worktrees/<agent-name>

# Delete the merged branch
git branch -d worktree-agent-<id>

# Final sweep: remove leftover directory
rm -rf .claude/worktrees/
```

<caution>
- Always merge the worktree branch before removing it.
- Always clean up worktrees before ending a session.
- Run `git worktree list` as a final check — only the main working tree should remain.
</caution>
