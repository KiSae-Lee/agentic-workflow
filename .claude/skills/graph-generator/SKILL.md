---
name: graph-generator
description: Analyzes source code to generate Graphviz diagrams and optionally a CODEBASE_MAP.md. Use for architecture visualization or codebase orientation.
---

# Graph Generator

Dispatch a subagent to analyze source code and generate Graphviz (DOT) diagrams.

**Announce at start**: "I'm using the graph-generator skill."

## Process

1. Identify the scope (which files/modules to analyze)
2. Dispatch a **general-purpose subagent** with the analysis prompt below
3. The subagent generates diagrams and optionally a CODEBASE_MAP.md
4. Report back with the generated artifacts

## Dispatch Prompt

Use the Agent tool (general-purpose type) with this prompt:

```
You are an expert Software Architect and Graphviz Specialist. Analyze the provided codebase and generate two types of Graphviz diagrams:

### 1. Procedure Explanation Diagram (Execution Flow)
- Map the exact function-call sequence with core functionality at each step
- Every function must appear as a node — no omissions
- Use directed graphs (`digraph`) with annotated edges
- If distinct execution paths exist (success/failure, branching), generate a SEPARATE digraph for each scenario

### 2. Function Explanation Diagram (Function Catalog)
- Standalone structural diagram listing all functions
- For each: Function Name, Role (Controller/Helper/Validator/etc.), Description
- Use HTML-like labels or Record shapes for tabular format

### Output
- Only valid Graphviz DOT code
- Each diagram in its own markdown ```dot code block
- Properly escaped strings

[SCOPE]: {files/modules to analyze}
```

## Optional: Generate CODEBASE_MAP.md

If invoked with `--map` or if the user requests a codebase map:

Add to the subagent prompt:

```
### 3. CODEBASE_MAP.md
Generate a slim orientation file at `CODEBASE_MAP.md` in the project root:
- Module tree showing directory structure
- One-line description per module/file explaining its role
- Keep under 100 lines total
- No implementation details, just "what lives where"
```

This replaces the expensive full CODEBASE.md with a lightweight alternative.
