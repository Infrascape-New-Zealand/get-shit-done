# Infrascape Get-Shit-Done Fork

This is the Infrascape fork of [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done). The upstream `gsd` namespace has been migrated to the `iscape` namespace with significant customizations.

## Repository Structure

```
commands/
├── .claude-plugin/marketplace.json   # Plugin registry
└── iscape/
    ├── .claude-plugin/plugin.json    # Plugin metadata
    ├── commands/                     # Slash commands (/iscape:*)
    ├── workflows/                    # Multi-step procedure references
    ├── references/                   # Principle/pattern documents
    ├── templates/                    # File templates
    └── agents/                       # Agent definitions (iscape-*.md)
```

## Merging Upstream Changes

When the upstream repo (gsd-build/get-shit-done) has new releases, use this procedure to bring changes into the iscape namespace.

### 1. Fetch upstream

```bash
git fetch https://github.com/gsd-build/get-shit-done.git main:refs/remotes/gsd-build/main
```

### 2. Identify new and changed files

Compare upstream gsd commands with current iscape commands:

```bash
# List upstream commands
git ls-tree --name-only gsd-build/main -- commands/gsd/ | sed 's|commands/gsd/||' | sort > /tmp/upstream_cmds.txt

# List current iscape commands
ls commands/iscape/commands/ | sort > /tmp/iscape_cmds.txt

# New in upstream (need to add)
comm -23 /tmp/upstream_cmds.txt /tmp/iscape_cmds.txt

# In iscape only (our additions, preserve these)
comm -13 /tmp/upstream_cmds.txt /tmp/iscape_cmds.txt
```

Repeat for workflows, references, templates, and agents:

```bash
# Workflows
git ls-tree --name-only gsd-build/main -- get-shit-done/workflows/ | sed 's|get-shit-done/workflows/||' | sort > /tmp/upstream_wf.txt
ls commands/iscape/workflows/ | sort > /tmp/iscape_wf.txt
comm -23 /tmp/upstream_wf.txt /tmp/iscape_wf.txt

# References
git ls-tree --name-only gsd-build/main -- get-shit-done/references/ | sed 's|get-shit-done/references/||' | sort

# Templates
git ls-tree -r --name-only gsd-build/main -- get-shit-done/templates/ | sed 's|get-shit-done/templates/||' | sort

# Agents
git ls-tree --name-only gsd-build/main -- agents/ | sed 's|agents/||' | sort
```

### 3. Extract and transform new files

For each new file category, extract from upstream and apply namespace transformations.

**Upstream path mapping:**

| Upstream | Iscape |
|----------|--------|
| `commands/gsd/*.md` | `commands/iscape/commands/*.md` |
| `get-shit-done/workflows/*.md` | `commands/iscape/workflows/*.md` |
| `get-shit-done/references/*.md` | `commands/iscape/references/*.md` |
| `get-shit-done/templates/*` | `commands/iscape/templates/*` |
| `agents/gsd-*.md` | `commands/iscape/agents/iscape-*.md` |

**Extract and transform pattern:**

```bash
# Example: extract a new workflow
git show gsd-build/main:get-shit-done/workflows/{filename} | sed \
  -e 's|/gsd:|/iscape:|g' \
  -e 's|gsd:|iscape:|g' \
  -e 's|\.planning/|context/|g' \
  -e 's|~/.claude/get-shit-done/|./|g' \
  -e 's|@~/.claude/get-shit-done/|@./|g' \
  -e 's| GSD | Iscape |g' \
  > commands/iscape/workflows/{filename}
```

### 4. Apply comprehensive namespace transformations

After initial extraction, run a second pass to catch all remaining `gsd` references:

```bash
find commands/iscape -name '*.md' -o -name '*.json' | xargs grep -l 'gsd' 2>/dev/null | while read f; do
  sed -i \
    -e 's|gsd-tools\.cjs|iscape-tools.cjs|g' \
    -e 's|gsd-tools|iscape-tools|g' \
    -e 's|gsd-project-researcher|iscape-project-researcher|g' \
    -e 's|gsd-research-synthesizer|iscape-research-synthesizer|g' \
    -e 's|gsd-roadmapper|iscape-roadmapper|g' \
    -e 's|gsd-planner|iscape-planner|g' \
    -e 's|gsd-executor|iscape-executor|g' \
    -e 's|gsd-verifier|iscape-verifier|g' \
    -e 's|gsd-debugger|iscape-debugger|g' \
    -e 's|gsd-integration-checker|iscape-integration-checker|g' \
    -e 's|gsd-plan-checker|iscape-plan-checker|g' \
    -e 's|gsd-codebase-mapper|iscape-codebase-mapper|g' \
    -e 's|gsd-phase-researcher|iscape-phase-researcher|g' \
    -e 's|~/\.gsd/|~/.iscape/|g' \
    -e 's|"gsd/phase-|"iscape/phase-|g' \
    -e 's|commands/gsd/|commands/iscape/commands/|g' \
    -e 's|`gsd new-project`|`iscape new-project`|g' \
    -e 's|`gsd <command>`|`iscape <command>`|g' \
    -e 's| gsd command| iscape command|g' \
    "$f"
done
```

### 4b. Fix subagent_type references

Upstream uses custom agent types (`gsd-executor`, `gsd-planner`, etc.) which become `iscape-executor`, `iscape-planner`, etc. after namespace transformation. Claude Code only supports built-in agent types, so all must be replaced with `general-purpose`:

```bash
find commands/iscape -name '*.md' | xargs sed -i \
  's/subagent_type="iscape-[^"]*"/subagent_type="general-purpose"/g'
```

### 5. Verify zero remaining gsd references

```bash
grep -r '\bgsd\b' commands/iscape/
```

This must return zero results. Fix any remaining references manually.

### 6. Install globally

```bash
cp -r commands/.claude-plugin ~/.claude/commands/.claude-plugin
cp -r commands/iscape ~/.claude/commands/iscape
```

### 7. Skip list

These upstream files are intentionally excluded:

- `commands/gsd/join-discord.md` - GSD community specific
- `commands/gsd/update.md` - GSD update mechanism (not applicable)
- `commands/gsd/reapply-patches.md` - GSD update mechanism
- `commands/gsd/new-project.md.bak` - Backup file
- `get-shit-done/workflows/update.md` - GSD update mechanism
- `get-shit-done/bin/` - gsd-tools CLI (iscape uses direct file operations)
- `hooks/` - GSD-specific hooks

### 8. Preserve list

These iscape-only files must not be overwritten or removed during merges:

**Commands:** `consider-issues.md`, `create-roadmap.md`, `discuss-milestone.md`, `execute-plan.md`, `plan-fix.md`

**Workflows:** `create-milestone.md`, `create-roadmap.md`, `discuss-milestone.md`

**References:** `plan-format.md`, `principles.md`, `research-pitfalls.md`, `scope-estimation.md`

**Templates:** `active_work.md`, `decision-entry.md`, `phase-plan.md`, `phase-summary.md`, `session-handoff.md`, `traceability.md`

**Agents:** `project_lead_agent/` directory

## Namespace Transformation Reference

| Pattern | Replacement |
|---------|-------------|
| `/gsd:` | `/iscape:` |
| `gsd:command` | `iscape:command` |
| `.planning/` | `context/` |
| `~/.claude/get-shit-done/` | `./` |
| `@~/.claude/get-shit-done/` | `@./` |
| `gsd-tools.cjs` | `iscape-tools.cjs` |
| `gsd-{agent}` | `iscape-{agent}` |
| `~/.gsd/` | `~/.iscape/` |
| `GSD` (branding) | `Iscape` |
| `commands/gsd/` | `commands/iscape/commands/` |
| `subagent_type="iscape-*"` | `subagent_type="general-purpose"` |

## Customization Notes

Existing iscape commands are **heavily customized** compared to upstream gsd equivalents. Key differences:

- **Inlined workflows**: iscape commands inline workflow steps rather than delegating to external workflow files
- **No gsd-tools dependency**: iscape uses direct file operations instead of the `gsd-tools.cjs` CLI
- **Generic agents**: iscape uses `general-purpose` subagent type instead of specialized `gsd-*` agent types
- **Flat path structure**: `context/` instead of `.planning/`, relative `@./` instead of absolute `~/.claude/get-shit-done/`
- **Explicit model profiles**: inline lookup tables instead of CLI-based resolution

Because of this, **do not overwrite existing iscape files** when merging upstream changes. Only add new files and selectively port significant new features from upstream into the customized iscape versions.
