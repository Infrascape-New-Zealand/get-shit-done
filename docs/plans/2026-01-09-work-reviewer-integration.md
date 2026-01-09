# Work-Reviewer Integration Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Modify plan generation to require tests in verify elements and trigger work-reviewer agent as blocking quality gate.

**Architecture:** Enhance the plan-phase workflow to enforce testable verify elements, add checkpoint:work-review as a new checkpoint type, and update execute-phase to spawn the work-reviewer agent when this checkpoint is reached.

**Tech Stack:** Markdown templates, XML task structure, Claude Code Task tool with work-reviewer subagent

---

## Task 1: Add checkpoint:work-review to checkpoints.md

**Files:**
- Modify: `commands/iscape/references/checkpoints.md`

**Step 1: Read the current file structure**

Understand where to insert the new checkpoint type (after checkpoint:human-action section, before </checkpoint_types>).

**Step 2: Add checkpoint:work-review section**

Insert after the `## checkpoint:human-action` section, before `</checkpoint_types>`:

```markdown
## checkpoint:work-review (quality gate)

**When:** All implementation tasks complete, before phase can be marked done.

**Purpose:** Spawn work-reviewer agent to verify all tasks were completed as designed using git evidence and test results.

**Structure:**
```xml
<task type="checkpoint:work-review" gate="blocking">
  <what-to-review>All tasks in this plan</what-to-review>
  <verification-commands>
    - npm run test
    - npm run build
  </verification-commands>
  <agent>work-reviewer</agent>
  <output>context/phases/XX-name/plans/YY-REVIEW.md</output>
  <resume-signal>
    - REVIEW.md shows all tasks ✅ Complete, OR
    - Human approves remediation, gaps are fixed, re-review passes
  </resume-signal>
</task>
```

**Behavior:**
1. Execution workflow reaches this checkpoint
2. Spawns work-reviewer agent with plan file path
3. Work-reviewer analyzes git history, runs verify commands from all tasks
4. Produces REVIEW.md with verification status for each task
5. If gaps exist → blocks, produces recommendations, awaits human approval
6. After remediation → re-runs work-reviewer until all tasks ✅

**Use for:** Every plan - this is mandatory, not optional.

**Do NOT skip:** Unlike other checkpoints, this one is always required at plan end.
```

**Step 3: Update summary section**

Find the `<summary>` section and add checkpoint:work-review to the priority list:

```markdown
**Checkpoint priority:**
1. **checkpoint:work-review** (mandatory) - Quality gate after all tasks, spawns work-reviewer agent
2. **checkpoint:human-verify** (90%) - Claude automated, human confirms visual/functional correctness
3. **checkpoint:decision** (9%) - Human makes architectural/technology choices
4. **checkpoint:human-action** (1%) - Truly unavoidable manual steps with no API/CLI
```

**Step 4: Commit**

```bash
git add commands/iscape/references/checkpoints.md
git commit -m "docs(checkpoints): add checkpoint:work-review specification

- New checkpoint type for work-reviewer agent integration
- Blocking quality gate that verifies all tasks via git evidence
- Produces REVIEW.md with verification status
- Mandatory at end of every plan"
```

---

## Task 2: Update plan-format.md with verify requirements

**Files:**
- Modify: `commands/iscape/references/plan-format.md`

**Step 1: Update the verify field specification**

Find the `<field name="verify">` section (around line 98-110) and replace with:

```markdown
<field name="verify">
**What it is**: Executable test command that proves the task is complete.

**Requirements:**
- Must reference a test file created as part of the task
- Must be a runnable command (not manual verification)
- Test file must be listed in the `<files>` element

**Good**:
- `npm run test -- tests/api/auth/login.test.ts`
- `pytest tests/test_user_registration.py -v`
- `go test ./internal/auth/... -run TestLogin`

**Bad**:
- "It works", "Looks good", "User can log in" (subjective)
- `curl -X POST /api/auth/login` (no test, just manual check)
- `npm test` (too broad, doesn't verify specific task)

The verify command output is used by work-reviewer to determine task completion status.
</field>
```

**Step 2: Update the task anatomy example**

Find the example task (around line 132-139) and update to include test file:

```xml
<task type="auto">
  <name>Task 3: Create login endpoint with JWT</name>
  <files>src/app/api/auth/login/route.ts, tests/api/auth/login.test.ts</files>
  <action>POST endpoint accepting {email, password}. Query User by email, compare password with bcrypt. On match, create JWT with jose library, set as httpOnly cookie (15-min expiry). Return 200. On mismatch, return 401. Also create test file covering valid/invalid credentials.</action>
  <verify>npm run test -- tests/api/auth/login.test.ts</verify>
  <done>Test passes: valid credentials → 200 + cookie, invalid → 401, missing fields → 400.</done>
</task>
```

**Step 3: Add checkpoint:work-review to task_types section**

Find the `<task_types>` section and add after checkpoint:decision:

```markdown
<type name="checkpoint:work-review">
**Mandatory quality gate** - Spawns work-reviewer agent to verify all tasks completed.

**Structure:**
```xml
<task type="checkpoint:work-review" gate="blocking">
  <what-to-review>All tasks in this plan</what-to-review>
  <verification-commands>
    - [test command from project]
    - [build command from project]
  </verification-commands>
  <agent>work-reviewer</agent>
  <output>context/phases/XX-name/plans/{phase}-{plan}-REVIEW.md</output>
  <resume-signal>REVIEW.md shows all tasks ✅ Complete, or human approves remediation</resume-signal>
</task>
```

**Required at end of every plan.** Work-reviewer uses git evidence and test output to verify each task.

See ./checkpoints.md for full checkpoint:work-review specification.
</type>
```

**Step 4: Update anti_patterns section**

Add to the `<unverifiable_completion>` section:

```markdown
- "npm test" (which tests? the task needs its own test file)
- "curl localhost:3000/api" (manual check, not a test)
```

**Step 5: Commit**

```bash
git add commands/iscape/references/plan-format.md
git commit -m "docs(plan-format): require tests in verify elements

- Verify field must reference test file created in task
- Files element must include test file path
- Action must include test creation
- Added checkpoint:work-review task type
- Updated examples to show test requirements"
```

---

## Task 3: Update plan-phase.md workflow

**Files:**
- Modify: `commands/iscape/workflows/plan-phase.md`

**Step 1: Add test requirement to break_into_tasks step**

Find `<step name="break_into_tasks">` and add after the task requirements list:

```markdown
**Verify element requirements:**
Every `type="auto"` task MUST have:
- A test file listed in `<files>` element
- Test creation mentioned in `<action>` element
- Test command in `<verify>` element that runs that specific test

This ensures work-reviewer can verify task completion via test output.

**Example transformation:**
- Before: `<verify>curl returns 200</verify>`
- After: `<verify>npm run test -- tests/api/login.test.ts</verify>` (with test file in `<files>` and test creation in `<action>`)
```

**Step 2: Add work-review checkpoint requirement to write_phase_prompt step**

Find `<step name="write_phase_prompt">` and add before the closing paragraph:

```markdown
**Mandatory work-review checkpoint:**
Every plan MUST end with a checkpoint:work-review task:

```xml
<task type="checkpoint:work-review" gate="blocking">
  <what-to-review>All tasks in this plan</what-to-review>
  <verification-commands>
    - [project test command, e.g., npm run test]
    - [project build command, e.g., npm run build]
  </verification-commands>
  <agent>work-reviewer</agent>
  <output>context/phases/XX-name/plans/{phase}-{plan}-REVIEW.md</output>
  <resume-signal>REVIEW.md shows all tasks ✅ Complete, or human approves remediation and gaps are fixed</resume-signal>
</task>
```

This is NOT optional. Every plan gets this checkpoint as its final task.
```

**Step 3: Update success_criteria**

Find `<success_criteria>` and add:

```markdown
- [ ] Every auto task has test file in `<files>`, test creation in `<action>`, test command in `<verify>`
- [ ] Plan ends with checkpoint:work-review task
```

**Step 4: Commit**

```bash
git add commands/iscape/workflows/plan-phase.md
git commit -m "feat(plan-phase): require tests and work-review checkpoint

- Auto tasks must include test file in files element
- Verify element must run specific test command
- Every plan must end with checkpoint:work-review
- Updated success criteria to enforce requirements"
```

---

## Task 4: Update phase-prompt.md template

**Files:**
- Modify: `get-shit-done/templates/phase-prompt.md`

**Step 1: Update the auto task example**

Find the `<task type="auto">` example (around line 43-49) and update:

```xml
<task type="auto">
  <name>Task 1: [Action-oriented name]</name>
  <files>path/to/implementation.ext, tests/path/to/test.ext</files>
  <action>[Specific implementation - what to do, how to do it, what to avoid and WHY. Include: "Create test file covering [specific behaviors]."]</action>
  <verify>[Test command that runs the specific test file, e.g., npm run test -- tests/path/to/test.ext]</verify>
  <done>[Measurable acceptance criteria - test passes with specific assertions]</done>
</task>
```

**Step 2: Add work-review checkpoint before closing </tasks>**

Find the `</tasks>` closing tag (around line 98) and insert before it:

```xml
<!-- MANDATORY: Every plan ends with work-review checkpoint -->
<task type="checkpoint:work-review" gate="blocking">
  <what-to-review>All tasks in this plan</what-to-review>
  <verification-commands>
    - [project test command]
    - [project build command]
  </verification-commands>
  <agent>work-reviewer</agent>
  <output>context/phases/XX-name/plans/{phase}-{plan}-REVIEW.md</output>
  <resume-signal>REVIEW.md shows all tasks ✅ Complete, or human approves remediation and gaps are fixed</resume-signal>
</task>
```

**Step 3: Add note in key_elements section**

Find `<key_elements>` and add:

```markdown
- Every auto task includes test file in `<files>` and test command in `<verify>`
- checkpoint:work-review is mandatory final task (spawns work-reviewer agent)
```

**Step 4: Update the good_examples section**

Find the good example and update task 2 to show test requirement:

```xml
<task type="auto">
  <name>Task 2: Create login API endpoint</name>
  <files>src/app/api/auth/login/route.ts, tests/api/auth/login.test.ts</files>
  <action>POST endpoint that accepts {email, password}, validates against User table using bcrypt, returns JWT in httpOnly cookie with 15-min expiry. Use jose library for JWT. Create test file covering valid credentials (200 + cookie), invalid credentials (401), and missing fields (400).</action>
  <verify>npm run test -- tests/api/auth/login.test.ts</verify>
  <done>All tests pass: valid credentials return 200 + cookie, invalid return 401, missing fields return 400</done>
</task>
```

And add the work-review checkpoint at the end of the example's tasks section.

**Step 5: Commit**

```bash
git add get-shit-done/templates/phase-prompt.md
git commit -m "feat(phase-prompt): add test requirements and work-review checkpoint

- Auto task template includes test file in files element
- Verify element shows test command pattern
- Added mandatory checkpoint:work-review as final task
- Updated good example to show test inclusion"
```

---

## Task 5: Update execute-phase.md to handle checkpoint:work-review

**Files:**
- Modify: `commands/iscape/workflows/execute-phase.md`

**Step 1: Add work-review handling to checkpoint_protocol step**

Find `<step name="checkpoint_protocol">` and add after the checkpoint:human-action handling:

```markdown
**For checkpoint:work-review (mandatory quality gate):**

```
════════════════════════════════════════
CHECKPOINT: Work Review
════════════════════════════════════════

Task [X] of [Y]: Quality Gate - Work Reviewer

Spawning work-reviewer agent to verify all tasks...

Plan: [plan file path]
Verification commands from plan:
- [command 1]
- [command 2]

[Work-reviewer agent output will appear here]
════════════════════════════════════════
```

**Execution protocol for checkpoint:work-review:**

1. **Spawn work-reviewer agent:**
   ```
   Use Task tool with subagent_type="work-reviewer":

   Prompt: "Review the implementation work for [plan path].

   Parse the plan file to extract:
   - All task names and their <verify> commands
   - The <verification-commands> from the work-review checkpoint

   For each task:
   1. Check git history for commits addressing this task
   2. Run the task's <verify> command
   3. Record result: ✅ Complete (code + test pass), ⚠️ Partial (code but test fail/missing), ❌ Missing (no evidence)

   Create REVIEW.md at [output path from checkpoint] with:
   - Verification status for each task
   - Git commit evidence
   - Test output summary
   - Recommendations for gaps (which agent to spawn)

   If all tasks ✅: Report success
   If any gaps: Report gaps and await human approval for remediation"
   ```

2. **Handle work-reviewer result:**

   **If all tasks ✅ Complete:**
   ```
   ════════════════════════════════════════
   WORK REVIEW: PASSED
   ════════════════════════════════════════

   All [N] tasks verified:
   ✅ Task 1: [name] - [commit hash]
   ✅ Task 2: [name] - [commit hash]
   ...

   REVIEW.md: [path]

   Continuing to completion...
   ════════════════════════════════════════
   ```
   Continue to next step (create_summary).

   **If gaps exist:**
   ```
   ════════════════════════════════════════
   WORK REVIEW: GAPS FOUND
   ════════════════════════════════════════

   Verification Status:
   ✅ Task 1: [name] - verified
   ⚠️ Task 2: [name] - tests failing
   ❌ Task 3: [name] - not implemented

   REVIEW.md: [path]

   Recommendations:
   1. Task 2 - Launch test-engineer to fix failing tests
   2. Task 3 - Launch implementation-engineer to implement

   Proceed with remediation? (yes / review REVIEW.md / stop)
   ════════════════════════════════════════
   ```

   Wait for user response.

   - **If "yes":** Spawn recommended agents, then re-run work-reviewer
   - **If "review REVIEW.md":** Display full REVIEW.md content
   - **If "stop":** Halt execution, do not create SUMMARY

3. **Remediation loop:**

   After remediation agents complete:
   - Re-spawn work-reviewer to verify fixes
   - Repeat until all tasks ✅ or user stops
   - Each re-review appends to REVIEW.md with timestamp
```

**Step 2: Update parse_segments step**

Find `<step name="parse_segments">` and add note about work-review:

```markdown
**Note on checkpoint:work-review:**
The work-review checkpoint is always the final task. It MUST execute in main context (not subagent) because:
- It spawns the work-reviewer agent
- It may require remediation loop with user interaction
- It produces REVIEW.md that gates completion

When parsing segments, treat checkpoint:work-review like checkpoint:decision - always route to main context.
```

**Step 3: Update success_criteria**

Find `<success_criteria>` and add:

```markdown
- Work-review checkpoint passed (all tasks ✅ in REVIEW.md)
- REVIEW.md created alongside SUMMARY.md
```

**Step 4: Commit**

```bash
git add commands/iscape/workflows/execute-phase.md
git commit -m "feat(execute-phase): add checkpoint:work-review handling

- Spawn work-reviewer agent when checkpoint reached
- Parse plan to extract verify commands for each task
- Handle pass/fail outcomes with remediation loop
- Block completion until all tasks verified
- Updated success criteria to require REVIEW.md"
```

---

## Verification

After all tasks complete:

```bash
# Verify all files were modified
git log --oneline -5

# Check that checkpoint:work-review appears in all relevant files
grep -r "checkpoint:work-review" commands/iscape/
grep -r "checkpoint:work-review" get-shit-done/

# Check that verify test requirement appears
grep -r "test file" commands/iscape/references/plan-format.md
grep -r "test file" get-shit-done/templates/phase-prompt.md
```

---

## Summary

This plan modifies 5 files to integrate work-reviewer:

1. **checkpoints.md** - Full specification of checkpoint:work-review
2. **plan-format.md** - Verify element requires test, new checkpoint type documented
3. **plan-phase.md** - Workflow enforces test in verify, appends work-review checkpoint
4. **phase-prompt.md** - Template includes test pattern and mandatory work-review
5. **execute-phase.md** - Handles work-review by spawning agent and managing remediation loop
