---
name: iscape:execute-phase
description: Execute all plans in a phase with wave-based parallelization
argument-hint: "<phase-number> [--wave N] [--gaps-only] [--interactive]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - TodoWrite
  - AskUserQuestion
---

<objective>
Execute all plans in a phase using wave-based parallel execution.

Orchestrator stays lean: discover plans, analyze dependencies, group into waves, spawn subagents, collect results. Each subagent loads the full execute-plan context and handles its own plan.

Optional wave filter:
- `--wave N` executes only Wave `N` for pacing, quota management, or staged rollout
- phase verification/completion still only happens when no incomplete plans remain after the selected wave finishes

Flag handling rule:
- The optional flags documented below are available behaviors, not implied active behaviors
- A flag is active only when its literal token appears in `$ARGUMENTS`
- If a documented flag is absent from `$ARGUMENTS`, treat it as inactive

Context budget: ~15% orchestrator, 100% fresh per subagent.
</objective>

<execution_context>
@./workflows/execute-phase.md
</execution_context>

<context>
Phase: $ARGUMENTS

**Available optional flags (documentation only — not automatically active):**
- `--wave N` — Execute only Wave `N` in the phase. Use when you want to pace execution or stay inside usage limits.
- `--gaps-only` — Execute only gap closure plans (plans with `gap_closure: true` in frontmatter). Use after verify-work creates fix plans.
- `--interactive` — Execute plans sequentially inline (no subagents) with user checkpoints between tasks. Lower token usage, pair-programming style. Best for small phases, bug fixes, and verification gaps.

**Active flags must be derived from `$ARGUMENTS`:**
- `--wave N` is active only if the literal `--wave` token is present in `$ARGUMENTS`
- `--gaps-only` is active only if the literal `--gaps-only` token is present in `$ARGUMENTS`
- `--interactive` is active only if the literal `--interactive` token is present in `$ARGUMENTS`

@context/ROADMAP.md
@context/STATE.md
</context>

<process>
0. **Worktree gate (mandatory)**

   Run before any other step. Hard block — not a warning.

   ```bash
   CURRENT_BRANCH=$(git branch --show-current 2>/dev/null || git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "unknown")
   ```

   **If `CURRENT_BRANCH` is `main` or `master`:**

   Read STATE.md for the expected worktree path:
   ```bash
   WORKTREE_PATH=$(grep 'worktree_path:' context/STATE.md 2>/dev/null | sed 's/worktree_path:[[:space:]]*//')
   ```

   **If `WORKTREE_PATH` is non-empty:** STOP. Output this error and exit immediately — do not continue to any other step:

   ```
   Error: execute-phase must run from the milestone worktree, not main.

   You are on:       main
   Expected worktree: {WORKTREE_PATH}

   Switch to the milestone worktree first:

     cd {WORKTREE_PATH}

   Then re-run:

     /iscape:execute-phase {N}
   ```

   **If `WORKTREE_PATH` is empty** (worktree creation failed during new-milestone, or project predates this feature):
   Print soft warning and continue:
   ```
   Warning: Running on main — no milestone worktree found in STATE.md. Continuing without isolation.
   ```

   **If `CURRENT_BRANCH` is NOT `main`/`master`:** proceed normally.

1. **Resolve Model Profile and Team Config**

   Read model profile for agent spawning:
   ```bash
   MODEL_PROFILE=$(cat context/config.json 2>/dev/null | grep -o '"model_profile"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "balanced")
   ```

   Default to "balanced" if not set.

   **Model lookup table:**

   | Agent | quality | balanced | budget |
   |-------|---------|----------|--------|
   | iscape-executor | opus | sonnet | sonnet |
   | iscape-developer | opus | sonnet | sonnet |
   | iscape-verifier | sonnet | sonnet | haiku |

   Store resolved models for use in Task/Agent calls below.

   Read team config:
   ```bash
   TEAM_ENABLED=$(cat context/config.json 2>/dev/null | grep -o '"team"' | head -1 | grep -c 'team' || echo "0")
   # Then read full team block: name_template, size, member_model
   TEAM_SIZE=$(cat context/config.json 2>/dev/null | grep -o '"size"[[:space:]]*:[[:space:]]*[0-9]*' | grep -o '[0-9]*$' || echo "3")
   # Check if team.enabled is true
   TEAM_NAME_TEMPLATE=$(cat context/config.json 2>/dev/null | grep -o '"name_template"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "{phase}-dev-team")
   ```

   Store for use in steps below.

2. **Validate phase exists**
   - Find phase directory matching argument
   - Count PLAN.md files
   - Error if no plans found

3. **Discover plans**
   - List all *-PLAN.md files in phase directory
   - Check which have *-SUMMARY.md (already complete)
   - If `--gaps-only`: filter to only plans with `gap_closure: true`
   - If `--wave N`: filter to only plans in that wave; skip plans in other waves
   - Build list of incomplete plans

4. **Group by wave**
   - Read `wave` from each plan's frontmatter
   - Group plans by wave number
   - Report wave structure to user

5. **Execute waves**
   If `--interactive` is active: execute plans sequentially inline (no subagents), presenting results between each plan for user review.

   **Otherwise, choose execution path based on team config:**

   **Path A — Standard wave execution** (`team.enabled: false` or team feature unavailable):

   For each wave in order:
   - Spawn `iscape-executor` for each plan in wave (parallel Task calls)
   - Wait for completion (Task blocks)
   - Verify SUMMARYs created
   - Proceed to next wave

   **Path B — Team execution** (`team.enabled: true`):

   1. Resolve team name from `name_template` (replace `{phase}` with current phase slug):
      ```
      TEAM_NAME = "{phase}-dev-team"  # e.g., "01-foundation-dev-team"
      ```

   2. Read all incomplete plan contents and STATE.md upfront (@ syntax doesn't cross agent boundaries):
      ```bash
      STATE_CONTENT=$(cat context/STATE.md)
      PLAN_01_CONTENT=$(cat "{plan_01_path}")
      # ... for each plan
      ```

   3. Call `TeamCreate`:
      ```
      TeamCreate(team_name="{team_name}", description="Execute phase {phase}: {phase_name}")
      ```

   4. Create one task per incomplete plan via `TaskCreate` (embed plan content in description):
      ```
      TaskCreate(
        title="Execute {plan-id}: {plan-objective}",
        description="Plan file: {plan_path}\n\nProject state:\n{state_content}\n\nPlan content:\n{plan_content}"
      )
      ```
      Create all tasks before spawning agents.

   5. Resolve developer model (from `team.member_model` or profile table for `iscape-developer`).

   6. Spawn `team.size` developer agents in a **single message** (parallel start):
      ```
      Agent(
        prompt="You are dev-1 on team {team_name}. Project directory: {cwd}\n\nRead ~/.claude/teams/{team_name}/config.json to find your team lead. Claim tasks from the task list and execute plans per your agent definition.",
        subagent_type="general-purpose",
        team_name="{team_name}",
        name="dev-1",
        model="{developer_model}"
      )
      Agent(
        prompt="You are dev-2 on team {team_name}. Project directory: {cwd}\n\n...",
        subagent_type="general-purpose",
        team_name="{team_name}",
        name="dev-2",
        model="{developer_model}"
      )
      # ... up to team.size agents
      ```

   7. **Wait** for developer completion messages (delivered automatically when developers send them). Track which plans have SUMMARY.md files created.

   8. **Handle escalations** from developers:
      - Checkpoint messages → relay to user, collect response, send back to developer via `SendMessage`
      - Architectural blockers → present to user, send decision back via `SendMessage`

   9. When all tasks are `completed` in task list:
      - Send shutdown to each developer: `SendMessage(to="dev-N", message={type: "shutdown_request"})`
      - Call `TeamDelete`

6. **Aggregate results**
   - Collect summaries from all plans
   - Report phase completion status

7. **Commit any orchestrator corrections**
   Check for uncommitted changes before verification:
   ```bash
   git status --porcelain
   ```

   **If changes exist:** Orchestrator made corrections between executor completions. Commit them:
   ```bash
   git add -u && git commit -m "fix({phase}): orchestrator corrections"
   ```

   **If clean:** Continue to verification.

8. **Verify phase goal**
   Check config: `WORKFLOW_VERIFIER=$(cat context/config.json 2>/dev/null | grep -o '"verifier"[[:space:]]*:[[:space:]]*[^,}]*' | grep -o 'true\|false' || echo "true")`

   **If `workflow.verifier` is `false`:** Skip to step 9 (treat as passed).

   **Otherwise:**
   - Spawn `iscape-verifier` subagent with phase directory and goal
   - Verifier checks must_haves against actual codebase (not SUMMARY claims)
   - Creates VERIFICATION.md with detailed report
   - Route by status:
     - `passed` → continue to step 9
     - `human_needed` → present items, get approval or feedback
     - `gaps_found` → present gaps, offer `/iscape:plan-phase {X} --gaps`

9. **Update roadmap and state**
   - Update ROADMAP.md, STATE.md

10. **Update requirements**
    Mark phase requirements as Complete:
    - Read ROADMAP.md, find this phase's `Requirements:` line (e.g., "AUTH-01, AUTH-02")
    - Read REQUIREMENTS.md traceability table
    - For each REQ-ID in this phase: change Status from "Pending" to "Complete"
    - Write updated REQUIREMENTS.md
    - Skip if: REQUIREMENTS.md doesn't exist, or phase has no Requirements line

11. **Commit phase completion**
    Check `COMMIT_PLANNING_DOCS` from config.json (default: true).
    If false: Skip git operations for context/ files.
    If true: Bundle all phase metadata updates in one commit:
    - Stage: `git add context/ROADMAP.md context/STATE.md`
    - Stage REQUIREMENTS.md if updated: `git add context/REQUIREMENTS.md`
    - Commit: `docs({phase}): complete {phase-name} phase`

12. **Offer next steps**
    - Route to next action (see `<offer_next>`)
</process>

<offer_next>
Output this markdown directly (not as a code block). Route based on status:

| Status | Route |
|--------|-------|
| `gaps_found` | Route C (gap closure) |
| `human_needed` | Present checklist, then re-route based on approval |
| `passed` + more phases | Route A (next phase) |
| `passed` + last phase | Route B (milestone complete) |

---

**Route A: Phase verified, more phases remain**

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 iscape ► PHASE {Z} COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {Z}: {Name}**

{Y} plans executed
Goal verified ✓

───────────────────────────────────────────────────────────────

## ▶ Next Up

**Phase {Z+1}: {Name}** — {Goal from ROADMAP.md}

/iscape:discuss-phase {Z+1} — gather context and clarify approach

<sub>/clear first → fresh context window</sub>

───────────────────────────────────────────────────────────────

**Also available:**
- /iscape:plan-phase {Z+1} — skip discussion, plan directly
- /iscape:verify-work {Z} — manual acceptance testing before continuing

───────────────────────────────────────────────────────────────

---

**Route B: Phase verified, milestone complete**

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 iscape ► MILESTONE COMPLETE 🎉
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**v1.0**

{N} phases completed
All phase goals verified ✓

───────────────────────────────────────────────────────────────

## ▶ Next Up

**Audit milestone** — verify requirements, cross-phase integration, E2E flows

/iscape:audit-milestone

<sub>/clear first → fresh context window</sub>

───────────────────────────────────────────────────────────────

**Also available:**
- /iscape:verify-work — manual acceptance testing
- /iscape:complete-milestone — skip audit, archive directly

───────────────────────────────────────────────────────────────

---

**Route C: Gaps found — need additional planning**

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 iscape ► PHASE {Z} GAPS FOUND ⚠
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {Z}: {Name}**

Score: {N}/{M} must-haves verified
Report: context/phases/{phase_dir}/{phase}-VERIFICATION.md

### What's Missing

{Extract gap summaries from VERIFICATION.md}

───────────────────────────────────────────────────────────────

## ▶ Next Up

**Plan gap closure** — create additional plans to complete the phase

/iscape:plan-phase {Z} --gaps

<sub>/clear first → fresh context window</sub>

───────────────────────────────────────────────────────────────

**Also available:**
- cat context/phases/{phase_dir}/{phase}-VERIFICATION.md — see full report
- /iscape:verify-work {Z} — manual testing before planning

───────────────────────────────────────────────────────────────

---

After user runs /iscape:plan-phase {Z} --gaps:
1. Planner reads VERIFICATION.md gaps
2. Creates plans 04, 05, etc. to close gaps
3. User runs /iscape:execute-phase {Z} again
4. Execute-phase runs incomplete plans (04, 05...)
5. Verifier runs again → loop until passed
</offer_next>

<wave_execution>
**Parallel spawning:**

Before spawning, read file contents. The `@` syntax does not work across Task() boundaries.

```bash
# Read each plan and STATE.md
PLAN_01_CONTENT=$(cat "{plan_01_path}")
PLAN_02_CONTENT=$(cat "{plan_02_path}")
PLAN_03_CONTENT=$(cat "{plan_03_path}")
STATE_CONTENT=$(cat context/STATE.md)
```

Spawn all plans in a wave with a single message containing multiple Task calls, with inlined content:

```
Task(prompt="Execute plan at {plan_01_path}\n\nPlan:\n{plan_01_content}\n\nProject state:\n{state_content}", subagent_type="general-purpose", model="{executor_model}")
Task(prompt="Execute plan at {plan_02_path}\n\nPlan:\n{plan_02_content}\n\nProject state:\n{state_content}", subagent_type="general-purpose", model="{executor_model}")
Task(prompt="Execute plan at {plan_03_path}\n\nPlan:\n{plan_03_content}\n\nProject state:\n{state_content}", subagent_type="general-purpose", model="{executor_model}")
```

All three run in parallel. Task tool blocks until all complete.

**No polling.** No background agents. No TaskOutput loops.
</wave_execution>

<checkpoint_handling>
Plans with `autonomous: false` have checkpoints. The execute-phase.md workflow handles the full checkpoint flow:
- Subagent pauses at checkpoint, returns structured state
- Orchestrator presents to user, collects response
- Spawns fresh continuation agent (not resume)

See `@./workflows/execute-phase.md` step `checkpoint_handling` for complete details.
</checkpoint_handling>

<deviation_rules>
During execution, handle discoveries automatically:

1. **Auto-fix bugs** - Fix immediately, document in Summary
2. **Auto-add critical** - Security/correctness gaps, add and document
3. **Auto-fix blockers** - Can't proceed without fix, do it and document
4. **Ask about architectural** - Major structural changes, stop and ask user

Only rule 4 requires user intervention.
</deviation_rules>

<commit_rules>
**Per-Task Commits:**

After each task completes:
1. Stage only files modified by that task
2. Commit with format: `{type}({phase}-{plan}): {task-name}`
3. Types: feat, fix, test, refactor, perf, chore
4. Record commit hash for SUMMARY.md

**Plan Metadata Commit:**

After all tasks in a plan complete:
1. Stage plan artifacts only: PLAN.md, SUMMARY.md
2. Commit with format: `docs({phase}-{plan}): complete [plan-name] plan`
3. NO code files (already committed per-task)

**Phase Completion Commit:**

After all plans in phase complete (step 7):
1. Stage: ROADMAP.md, STATE.md, REQUIREMENTS.md (if updated), VERIFICATION.md
2. Commit with format: `docs({phase}): complete {phase-name} phase`
3. Bundles all phase-level state updates in one commit

**NEVER use:**
- `git add .`
- `git add -A`
- `git add src/` or any broad directory

**Always stage files individually.**
</commit_rules>

<success_criteria>
- [ ] All incomplete plans in phase executed
- [ ] Each plan has SUMMARY.md
- [ ] Phase goal verified (must_haves checked against codebase)
- [ ] VERIFICATION.md created in phase directory
- [ ] STATE.md reflects phase completion
- [ ] ROADMAP.md updated
- [ ] REQUIREMENTS.md updated (phase requirements marked Complete)
- [ ] User informed of next steps
</success_criteria>
