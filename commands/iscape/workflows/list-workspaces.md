<purpose>
List all Iscape workspaces found in ~/iscape-workspaces/ with their status.
</purpose>

<required_reading>
Read all files referenced by the invoking prompt's execution_context before starting.
</required_reading>

<process>

## 1. Setup

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/iscape-tools.cjs" init list-workspaces)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

Parse JSON for: `workspace_base`, `workspaces`, `workspace_count`.

## 2. Display

**If `workspace_count` is 0:**

```
No workspaces found in ~/iscape-workspaces/

Create one with:
  /iscape:new-workspace --name my-workspace --repos repo1,repo2
```

Done.

**If workspaces exist:**

Display a table:

```
Iscape Workspaces (~/iscape-workspaces/)

| Name | Repos | Strategy | Iscape Project |
|------|-------|----------|-------------|
| feature-a | 3 | worktree | Yes |
| feature-b | 2 | clone | No |

Manage:
  cd ~/iscape-workspaces/<name>     # Enter a workspace
  /iscape:remove-workspace <name>   # Remove a workspace
```

For each workspace, show:
- **Name** — directory name
- **Repos** — count from init data
- **Strategy** — from WORKSPACE.md
- **Iscape Project** — whether `context/PROJECT.md` exists (Yes/No)

</process>
