---
name: iscape:sketch
description: Sketch UI/design ideas with throwaway HTML mockups, or propose what to sketch next (frontier mode)
argument-hint: "[design idea to explore] [--quick] [--text] [--wrap-up] or [frontier]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
  - AskUserQuestion
  - WebSearch
  - WebFetch
  - mcp__context7__resolve-library-id
  - mcp__context7__query-docs
---
<objective>
Explore design directions through throwaway HTML mockups before committing to implementation.
Each sketch produces 2-3 variants for comparison. Sketches live in `context/sketches/` and
integrate with Iscape commit patterns, state tracking, and handoff workflows. Loads spike
findings to ground mockups in real data shapes and validated interaction patterns.

Two modes:
- **Idea mode** (default) — describe a design idea to sketch
- **Frontier mode** (no argument or "frontier") — analyzes existing sketch landscape and proposes consistency and frontier sketches

Does not require `/iscape-new-project` — auto-creates `context/sketches/` if needed.
</objective>

<execution_context>
@./workflows/sketch.md
@./references/ui-brand.md
@./references/sketch-theme-system.md
@./references/sketch-interactivity.md
@./references/sketch-tooling.md
@./references/sketch-variant-patterns.md
</execution_context>

<runtime_note>
**Copilot (VS Code):** Use `vscode_askquestions` wherever this workflow calls `AskUserQuestion`.
</runtime_note>

<context>
Design idea: $ARGUMENTS

**Available flags:**
- `--quick` — Skip mood/direction intake, jump straight to decomposition and building. Use when the design direction is already clear.
- `--wrap-up` — Package sketch design findings into a persistent project skill for future build conversations. Runs the sketch-wrap-up workflow.
</context>

<process>
Execute the sketch workflow from @./workflows/sketch.md end-to-end.
Preserve all workflow gates (intake, decomposition, target stack research, variant evaluation, MANIFEST updates, commit patterns).
</process>
