---
name: iscape:list-workspaces
description: List active Iscape workspaces and their status
allowed-tools:
  - Bash
  - Read
---
<objective>
Scan `~/iscape-workspaces/` for workspace directories containing `WORKSPACE.md` manifests. Display a summary table with name, path, repo count, strategy, and Iscape project status.
</objective>

<execution_context>
@./workflows/list-workspaces.md
@./references/ui-brand.md
</execution_context>

<process>
Execute the list-workspaces workflow from @./workflows/list-workspaces.md end-to-end.
</process>
