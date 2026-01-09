# Work-Reviewer Integration Design

## Overview

Enhance plan generation to ensure all tasks include verifiable tests and trigger the work-reviewer agent as a blocking quality gate after implementation completes.

## Requirements

1. **Test in verify elements** - Every task's `<verify>` element must include a runnable test that proves completion
2. **Work-reviewer trigger** - Plans include a final `checkpoint:work-review` task that spawns the work-reviewer agent
3. **Blocking behavior** - Phase cannot complete until work-reviewer passes (all tasks ✅) or human approves remediation

## Design

### Verify Element Enhancement

Every `<verify>` element must reference a test command. The `<files>` element includes test files, and `<action>` includes test creation:

```xml
<task type="auto">
  <name>Create login endpoint with JWT</name>
  <files>src/app/api/auth/login/route.ts, tests/api/auth/login.test.ts</files>
  <action>POST endpoint accepting {email, password}. Also create test file covering valid/invalid credentials.</action>
  <verify>npm run test -- tests/api/auth/login.test.ts</verify>
  <done>Test passes: valid credentials → 200 + cookie, invalid → 401, missing fields → 400</done>
</task>
```

### Work-Review Checkpoint Task

New checkpoint type as the final task in every plan:

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

### Execution Flow

1. Implementation tasks execute (each creating tests as part of implementation)
2. Execution workflow reaches `checkpoint:work-review`
3. Work-reviewer agent spawns with plan file path
4. Agent analyzes git history, runs verify commands, produces REVIEW.md
5. If gaps exist → blocks, produces recommendations, awaits human approval
6. After remediation → re-runs work-reviewer until all tasks ✅

## Files to Modify

| File | Change |
|------|--------|
| `commands/iscape/references/plan-format.md` | Add verify element requirements (must include test), document `checkpoint:work-review` task type |
| `commands/iscape/references/checkpoints.md` | Add `checkpoint:work-review` section with full specification |
| `commands/iscape/workflows/plan-phase.md` | Add step requiring test in verify elements, add step to append work-review checkpoint |
| `get-shit-done/templates/phase-prompt.md` | Add work-review checkpoint as mandatory final task in template |
| `commands/iscape/workflows/execute-phase.md` | Add handling for `checkpoint:work-review` to spawn work-reviewer agent |

## Work-Reviewer Agent Location

The work-reviewer agent definition should be copied to the project using iscape, or referenced from a shared location. Current reference implementation: `/home/mathew/sources/ComplianceFlow/.claude/agents/work-reviewer.md`
