<planning_config>

Configuration options for `context/` directory behavior.

<config_schema>
```json
"planning": {
  "commit_docs": true,
  "search_gitignored": false
},
"team": {
  "enabled": false,
  "name_template": "{phase}-dev-team",
  "size": 3,
  "member_model": null
},
"git": {
  "branching_strategy": "none",
  "use_worktree": false,
  "pr_on_complete": false,
  "phase_branch_template": "iscape/phase-{phase}-{slug}",
  "milestone_branch_template": "iscape/{milestone}-{slug}"
}
```

| Option | Default | Description |
|--------|---------|-------------|
| `commit_docs` | `true` | Whether to commit planning artifacts to git |
| `search_gitignored` | `false` | Add `--no-ignore` to broad rg searches |
| `git.branching_strategy` | `"none"` | Git branching approach: `"none"`, `"phase"`, or `"milestone"` |
| `git.use_worktree` | `false` | Create isolated git worktree for milestone; all commits go to milestone branch, not main |
| `git.pr_on_complete` | `false` | Create GitHub PR from milestone branch → main when `complete-milestone` runs |
| `git.phase_branch_template` | `"iscape/phase-{phase}-{slug}"` | Branch template for phase strategy |
| `git.milestone_branch_template` | `"iscape/{milestone}-{slug}"` | Branch template for milestone/worktree strategy |
| `team.enabled` | `false` | Use Agent Teams for parallel execution (requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`) |
| `team.name_template` | `"{phase}-dev-team"` | Team name pattern; `{phase}` replaced with phase slug |
| `team.size` | `3` | Number of developer agents to spawn in the team |
| `team.member_model` | `null` | Model override for all developer agents (null = profile default for `iscape-developer`) |
</config_schema>

<commit_docs_behavior>

**When `commit_docs: true` (default):**
- Planning files committed normally
- SUMMARY.md, STATE.md, ROADMAP.md tracked in git
- Full history of planning decisions preserved

**When `commit_docs: false`:**
- Skip all `git add`/`git commit` for `context/` files
- User must add `context/` to `.gitignore`
- Useful for: OSS contributions, client projects, keeping planning private

**Using direct file operations (preferred):**

```bash
# Check commit_docs config before committing:
COMMIT_DOCS=$(cat context/config.json 2>/dev/null | grep -o '"commit_docs"[[:space:]]*:[[:space:]]*[a-z]*' | grep -o '[a-z]*$' || echo "true")
GITIGNORED=false
git check-ignore -q context/ 2>/dev/null && GITIGNORED=true

if [ "$COMMIT_DOCS" = "true" ] && [ "$GITIGNORED" = "false" ]; then
  git add context/STATE.md
  git commit -m "docs: update state"
fi
```

**Auto-detection:** If `context/` is gitignored, `commit_docs` is automatically `false` regardless of config.json. This prevents git errors when users have `context/` in `.gitignore`.

**Commit with checks:**

```bash
# Check both config and gitignore status before committing
COMMIT_DOCS=$(cat context/config.json 2>/dev/null | grep -o '"commit_docs"[[:space:]]*:[[:space:]]*[a-z]*' | grep -o '[a-z]*$' || echo "true")
git check-ignore -q context/ 2>/dev/null && COMMIT_DOCS="false"

if [ "$COMMIT_DOCS" = "true" ]; then
  git add context/STATE.md
  git commit -m "docs: update state"
fi
```

</commit_docs_behavior>

<search_behavior>

**When `search_gitignored: false` (default):**
- Standard rg behavior (respects .gitignore)
- Direct path searches work: `rg "pattern" context/` finds files
- Broad searches skip gitignored: `rg "pattern"` skips `context/`

**When `search_gitignored: true`:**
- Add `--no-ignore` to broad rg searches that should include `context/`
- Only needed when searching entire repo and expecting `context/` matches

**Note:** Most Iscape operations use direct file reads or explicit paths, which work regardless of gitignore status.

</search_behavior>

<setup_uncommitted_mode>

To use uncommitted mode:

1. **Set config:**
   ```json
   "planning": {
     "commit_docs": false,
     "search_gitignored": true
   }
   ```

2. **Add to .gitignore:**
   ```
   context/
   ```

3. **Existing tracked files:** If `context/` was previously tracked:
   ```bash
   git rm -r --cached context/
   git commit -m "chore: stop tracking planning docs"
   ```

4. **Branch merges:** When using `branching_strategy: phase` or `milestone`, the `complete-milestone` workflow automatically strips `context/` files from staging before merge commits when `commit_docs: false`.

</setup_uncommitted_mode>

<branching_strategy_behavior>

**Branching Strategies:**

| Strategy | When branch created | Branch scope | Merge point |
|----------|---------------------|--------------|-------------|
| `none` | Never | N/A | N/A |
| `phase` | At `execute-phase` start | Single phase | User merges after phase |
| `milestone` | At first `execute-phase` of milestone | Entire milestone | At `complete-milestone` |

**When `git.branching_strategy: "none"` (default):**
- All work commits to current branch
- Standard Iscape behavior

**When `git.branching_strategy: "phase"`:**
- `execute-phase` creates/switches to a branch before execution
- Branch name from `phase_branch_template` (e.g., `iscape/phase-03-authentication`)
- All plan commits go to that branch
- User merges branches manually after phase completion
- `complete-milestone` offers to merge all phase branches

**When `git.branching_strategy: "milestone"`:**
- First `execute-phase` of milestone creates the milestone branch
- Branch name from `milestone_branch_template` (e.g., `iscape/v1.0-mvp`)
- All phases in milestone commit to same branch
- `complete-milestone` offers to merge milestone branch to main

**Template variables:**

| Variable | Available in | Description |
|----------|--------------|-------------|
| `{phase}` | phase_branch_template | Zero-padded phase number (e.g., "03") |
| `{slug}` | Both | Lowercase, hyphenated name |
| `{milestone}` | milestone_branch_template | Milestone version (e.g., "v1.0") |

**Checking the config:**

Read `context/config.json` directly to get branching config:
```bash
# Read config values
BRANCHING_STRATEGY=$(cat context/config.json 2>/dev/null | grep -o '"branching_strategy"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "none")
PHASE_BRANCH_TEMPLATE=$(cat context/config.json 2>/dev/null | grep -o '"phase_branch_template"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "iscape/phase-{phase}-{slug}")
MILESTONE_BRANCH_TEMPLATE=$(cat context/config.json 2>/dev/null | grep -o '"milestone_branch_template"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "iscape/{milestone}-{slug}")
```

**Branch creation:**

```bash
# For phase strategy
if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  PHASE_SLUG=$(echo "$PHASE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$PHASE_BRANCH_TEMPLATE" | sed "s/{phase}/$PADDED_PHASE/g" | sed "s/{slug}/$PHASE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi

# For milestone strategy
if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  MILESTONE_SLUG=$(echo "$MILESTONE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BRANCH_NAME=$(echo "$MILESTONE_BRANCH_TEMPLATE" | sed "s/{milestone}/$MILESTONE_VERSION/g" | sed "s/{slug}/$MILESTONE_SLUG/g")
  git checkout -b "$BRANCH_NAME" 2>/dev/null || git checkout "$BRANCH_NAME"
fi
```

**Merge options at complete-milestone:**

| Option | Git command | Result |
|--------|-------------|--------|
| Squash merge (recommended) | `git merge --squash` | Single clean commit per branch |
| Merge with history | `git merge --no-ff` | Preserves all individual commits |
| Delete without merging | `git branch -D` | Discard branch work |
| Keep branches | (none) | Manual handling later |

Squash merge is recommended — keeps main branch history clean while preserving the full development history in the branch (until deleted).

**Use cases:**

| Strategy | Best for |
|----------|----------|
| `none` | Solo development, simple projects |
| `phase` | Code review per phase, granular rollback, team collaboration |
| `milestone` | Release branches, staging environments, PR per version |

</branching_strategy_behavior>

<team_config>

## Dev Team Configuration

When `team.enabled: true`, `execute-phase` creates a named Agent Team and spawns parallel developer agents who claim and execute plans from a shared task queue.

**Requirements:**
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` must be set in Claude Code settings env
- Team feature enabled in Claude Code session

**How it works:**
1. `execute-phase` calls `TeamCreate` with a team name derived from `team.name_template`
2. One `TaskCreate` per incomplete plan (plan content embedded in task description)
3. `team.size` developer agents spawned in parallel — each claims tasks by lowest ID
4. Developers execute plans using the full `iscape-executor` workflow
5. Team lead receives completion messages, then calls `TeamDelete`

**Team name template variables:**
| Variable | Description | Example |
|----------|-------------|---------|
| `{phase}` | Phase slug from directory name | `01-foundation` |
| `{milestone}` | Milestone version | `v1.0` |

**Example config for 2 parallel developers:**
```json
{
  "team": {
    "enabled": true,
    "name_template": "{phase}-dev-team",
    "size": 2,
    "member_model": null
  }
}
```

**Fallback behavior:** If team execution fails (agent teams not enabled), falls back to standard wave-based Task spawning automatically.

</team_config>

<worktree_config>

## Worktree & PR Configuration

When `git.use_worktree: true`, all milestone execution happens in an isolated git worktree. Main branch is never modified until the milestone PR is merged.

**Requirements:**
- Git 2.5+ (git worktree support)
- GitHub CLI (`gh`) for PR creation when `pr_on_complete: true`

**Workflow:**
1. First `execute-phase` of milestone → creates worktree at `../{project}-{milestone-slug}/`
2. All phases execute inside that worktree
3. `complete-milestone` → pushes branch, creates PR, removes worktree

**Works with or without team mode.** Combine both for maximum parallelism with PR-based review:
```json
{
  "team": { "enabled": true, "size": 3 },
  "git": { "use_worktree": true, "pr_on_complete": true, "branching_strategy": "milestone" }
}
```

</worktree_config>

</planning_config>
