## Workflow

### Project Design Workflow

```dot
digraph G {
    rankdir=TB;
    node [shape=box];

    "Dose PROJECT_DESIGN.md exist?" [shape=diamond]
    "Revision?" [shape=diamond]
    "Is related to scientific stuff?" [shape=diamond]
    "Architecture review needed?" [shape=diamond]

    "PROJECT Idea" -> "Dose PROJECT_DESIGN.md exist?"
    "Dose PROJECT_DESIGN.md exist?" -> "Start brainstorm based on PROJECT/Revision idea" [label=No]
    "Start brainstorm based on PROJECT/Revision idea" -> "Create/Update PROJECT_DESIGN.md"
    "Create/Update PROJECT_DESIGN.md" -> "Architecture review needed?"
    "Architecture review needed?" -> "Start arch-review" [label=Yes]
    "Start arch-review" -> "Create ARCH-REVIEW.md"
    "Create ARCH-REVIEW.md" -> "Create/Update PROJECT_DESIGN.md" [label=Update]
    "Architecture review needed?" -> "Is related to scientific stuff?" [label=No]
    "Is related to scientific stuff?" -> "Create/Update TODO.md" [label=No]

    "Is related to scientific stuff?" -> "Start sci-review" [label=Yes]
    "Start sci-review" -> "Create SCI-REVIEW.md"
    "Create SCI-REVIEW.md" -> "Create/Update PROJECT_DESIGN.md" [label=Update]


    "Create/Update TODO.md" -> "Revision?"
    "Revision?" -> "Start brainstorm based on PROJECT/Revision idea" [label="Yes. Ask for revision idea"]
    "Revision?" -> "End of session" [label=No]

    "Dose PROJECT_DESIGN.md exist?" -> "Read PROJECT_DESIGN.md" [label=Yes]
    "Read PROJECT_DESIGN.md" -> "Start brainstorm based on PROJECT/Revision idea"
}
```

<caution>
- Ask if you are confused the idea is belong to the `Project Design` or `Implementation`.
- `PROJECT_DESIGN.md` should include full-suite development plan and MVP development plan.
- `TODO.md` should clearify which item is belong to the MVP.
</caution>

### Implementation Workflow

```dot
digraph G {
rankdir=TB;
node [shape=box];

    "Does CODEBASE_MAP.md exist?" [shape=diamond]
    "Revision for the design?" [shape=diamond]
    "Revision for the plan?" [shape=diamond]
    "Algorithmic/scientific content?" [shape=diamond]
    "Any Critical or Important issue?" [shape=diamond]
    "Continue?" [shape=diamond]
    "Review?" [shape=diamond]

    "Implementation Idea" -> "Does CODEBASE_MAP.md exist?"
    subgraph cluster_1{
        style=dashed
        label=Explore

        "Does CODEBASE_MAP.md exist?" -> "Read CODEBASE_MAP.md and skip exploring" [label=Yes]
        "Does CODEBASE_MAP.md exist?" -> "Create CODEBASE_MAP.md (Using graph-generator skill --map)" [label=No]
        "Create CODEBASE_MAP.md (Using graph-generator skill --map)" -> "Read CODEBASE_MAP.md and skip exploring"

    }

    subgraph cluster_3{
        style=dashed
        label="Create specification"

        "Read CODEBASE_MAP.md and skip exploring" -> "Use brainstorming skill"
        "Use brainstorming skill" -> "DESIGN.md"
        "DESIGN.md" -> "Revision for the design?"
        "Revision for the design?" -> "Use arch-review skill" [label=No]
        "Use arch-review skill" -> "ARCH-REVIEW.md"
        "ARCH-REVIEW.md" -> "Use planning skill"
        "Use planning skill" -> "PLAN.md, TASK.md, NEXT_STEP.md"
        "PLAN.md, TASK.md, NEXT_STEP.md" -> "Revision for the plan?"
    }

    subgraph cluster_4{
        style=dashed
        label="Sci-Review (conditional)"

        "Revision for the plan?" -> "Algorithmic/scientific content?" [label=No]
        "Algorithmic/scientific content?" -> "Use sci-review skill" [label=Yes]
        "Use sci-review skill" -> "SCI-REVIEW.md"
        "SCI-REVIEW.md" -> "Use subagent-driven-development skill"
    }

    subgraph cluster_2 {
        style=dashed
        label="Implementation"

        "Algorithmic/scientific content?" -> "Use subagent-driven-development skill" [label=No]
        "Use subagent-driven-development skill" -> "Review?"
        "Review?" -> "Use code-review skill (subagent)" [label=Yes]
        "Review?" -> "IMPLEMENTATION_REPORT.md" [label=No]
        "Use code-review skill (subagent)" -> "CODE-REVIEW.md"
        "CODE-REVIEW.md" -> "Any Critical or Important issue?"
        "Any Critical or Important issue?" -> "Continue?" [label=Yes]
        "Continue?" -> "Use planning skill" [label=Yes]
        "Continue?" -> "IMPLEMENTATION_REPORT.md" [label=No]
        "Any Critical or Important issue?" -> "IMPLEMENTATION_REPORT.md" [label=No]
        "IMPLEMENTATION_REPORT.md" -> "Update CODEBASE_MAP.md"
    }

    "Revision for the design?" -> "End of session" [label=Yes]
    "Revision for the plan?" -> "End of session" [label=Yes]
    "Update CODEBASE_MAP.md" -> "Update pipelines/ (Using graph-generator skill)"
    "Update pipelines/ (Using graph-generator skill)" -> "Update TODO.md"
    "Update TODO.md" -> "End of session"
}
```

<caution>
- Remove the worktree if it is created for parallel tasks after the implmentation.
</caution>
