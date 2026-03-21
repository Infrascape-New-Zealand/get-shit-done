---
type: prompt
name: iscape:complete-milestone
description: Archive completed milestone and prepare for next version
argument-hint: <version>
allowed-tools:
  - Read
  - Write
  - Bash
---

<objective>
Mark milestone {{version}} complete, archive to milestones/, and update ROADMAP.md.

Purpose: Create historical record of shipped version, collapse completed work in roadmap, and prepare for next milestone.
Output: Milestone archived, roadmap reorganized, git tagged.
</objective>

<execution_context>
**Load these files NOW (before proceeding):**

- @./workflows/complete-milestone.md (main workflow)
- @./templates/milestone-archive.md (archive template)
  </execution_context>

<context>
**Project files:**
- `context/ROADMAP.md`
- `context/STATE.md`
- `context/PROJECT.md`

**User input:**

- Version: {{version}} (e.g., "1.0", "1.1", "2.0")
  </context>

<process>

**Follow complete-milestone.md workflow:**

1. **Verify readiness:**

   - Check all phases in milestone have completed plans (SUMMARY.md exists)
   - Present milestone scope and stats
   - Wait for confirmation

2. **Gather stats:**

   - Count phases, plans, tasks
   - Calculate git range, file changes, LOC
   - Extract timeline from git log
   - Present summary, confirm

3. **Extract accomplishments:**

   - Read all phase SUMMARY.md files in milestone range
   - Extract 4-6 key accomplishments
   - Present for approval

4. **Archive milestone:**

   - Create `context/milestones/v{{version}}-ROADMAP.md`
   - Extract full phase details from ROADMAP.md
   - Fill milestone-archive.md template
   - Update ROADMAP.md to one-line summary with link
   - Offer to create next milestone

5. **Update PROJECT.md:**

   - Add "Current State" section with shipped version
   - Add "Next Milestone Goals" section
   - Archive previous content in `<details>` (if v1.1+)

6. **Commit and tag:**

   - Stage: MILESTONES.md, PROJECT.md, ROADMAP.md, STATE.md, archive file
   - Commit: `chore: archive v{{version}} milestone`
   - Tag: `git tag -a v{{version}} -m "[milestone summary]"`
   - Ask about pushing tag

6.5. **Create PR** (when `git.pr_on_complete: true` in config.json):

   ```bash
   PR_ON_COMPLETE=$(cat context/config.json 2>/dev/null | grep -o '"pr_on_complete"[[:space:]]*:[[:space:]]*[a-z]*' | grep -o '[a-z]*$' || echo "false")
   ```

   If `true`:
   1. Verify `gh` CLI is available: `which gh`
   2. Identify milestone branch (from `git.milestone_branch_template` or current branch if in worktree)
   3. Push branch to remote:
      ```bash
      git push -u origin "{milestone-branch}"
      ```
   4. Create PR:
      ```bash
      gh pr create \
        --title "feat: {milestone-name} v{{version}}" \
        --base main \
        --head "{milestone-branch}" \
        --body "$(cat <<'EOF'
      ## Milestone v{{version}}: {milestone-name}

      ### Accomplishments
      {4-6 key accomplishments from step 3}

      ### Phases Completed
      {phase list with goals}

      ### Stats
      {plans executed, git tag}

      🤖 Generated with [iscape](https://github.com/infrascape/get-shit-done)
      EOF
      )"
      ```
   5. Present PR URL to user.

7. **Worktree cleanup** (when `git.use_worktree: true` in config.json):

   ```bash
   USE_WORKTREE=$(cat context/config.json 2>/dev/null | grep -o '"use_worktree"[[:space:]]*:[[:space:]]*[a-z]*' | grep -o '[a-z]*$' || echo "false")
   ```

   If `true`:
   1. Identify worktree path: `git worktree list` (find entry for milestone branch)
   2. Present to user:
      ```
      Worktree: {worktree-path}
      Branch:   {milestone-branch}
      PR:       {pr-url or "push and create PR when ready"}

      Remove worktree now? (you can still merge the PR after removal)
      ```
   3. If user confirms:
      ```bash
      # Ensure we're not inside the worktree before removing
      cd "{main-repo-path}"
      git worktree remove "{worktree-path}"
      ```
   4. Inform: "Worktree removed. Merge the PR on GitHub when ready."

8. **Offer next steps:**
   - Plan next milestone
   - Archive planning
   - Done for now

</process>

<success_criteria>

- Milestone archived to `context/milestones/v{{version}}-ROADMAP.md`
- ROADMAP.md collapsed to one-line entry
- PROJECT.md updated with current state
- Git tag v{{version}} created
- Commit successful
- PR created (if `git.pr_on_complete: true`) from milestone branch → main
- Worktree removed (if `git.use_worktree: true` and user confirmed)
- User knows next steps
  </success_criteria>

<critical_rules>

- **Load workflow first:** Read complete-milestone.md before executing
- **Verify completion:** All phases must have SUMMARY.md files
- **User confirmation:** Wait for approval at verification gates
- **Archive before collapsing:** Always create archive file before updating ROADMAP.md
- **One-line summary:** Collapsed milestone in ROADMAP.md should be single line with link
- **Context efficiency:** Archive keeps ROADMAP.md constant size
  </critical_rules>
