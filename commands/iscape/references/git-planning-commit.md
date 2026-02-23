# Git Planning Commit

Commit planning artifacts using direct git operations, checking `commit_docs` config and gitignore status.

## Commit Planning Files

Always check `commit_docs` config and gitignore status before committing `context/` files:

```bash
# Check if context/ is gitignored
git check-ignore -q context/ 2>/dev/null && GITIGNORED=true || GITIGNORED=false

# Check commit_docs config
COMMIT_DOCS=$(cat context/config.json 2>/dev/null | grep -o '"commit_docs"[[:space:]]*:[[:space:]]*[a-z]*' | grep -o '[a-z]*$' || echo "true")

# Skip if gitignored or commit_docs is false
if [ "$GITIGNORED" = "true" ] || [ "$COMMIT_DOCS" = "false" ]; then
  echo "Skipping planning commit (commit_docs=$COMMIT_DOCS, gitignored=$GITIGNORED)"
else
  git add context/STATE.md context/ROADMAP.md
  git commit -m "docs({scope}): {description}"
fi
```

## Amend previous commit

To fold `context/` file changes into the previous commit:

```bash
git add context/codebase/*.md
git commit --amend --no-edit
```

## Commit Message Patterns

| Command | Scope | Example |
|---------|-------|---------|
| plan-phase | phase | `docs(phase-03): create authentication plans` |
| execute-phase | phase | `docs(phase-03): complete authentication phase` |
| new-milestone | milestone | `docs: start milestone v1.1` |
| remove-phase | chore | `chore: remove phase 17 (dashboard)` |
| insert-phase | phase | `docs: insert phase 16.1 (critical fix)` |
| add-phase | phase | `docs: add phase 07 (settings page)` |

## When to Skip

- `commit_docs: false` in config
- `context/` is gitignored
- No changes to commit (check with `git status --porcelain context/`)
