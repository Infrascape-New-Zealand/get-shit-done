# Session Handoff Template

Template for `context/session_handoffs/YYYY-MM-DD-[task-name].md` — pause/resume context.

<template>

# Session Handoff: [Task Name]

**Created:** [YYYY-MM-DD HH:MM]
**Milestone:** [vX.X]
**Phase:** [XX-name]
**Plan:** [if applicable]

## Current State

**Status:** [Paused mid-task / Paused between tasks / Blocked]
**Last completed:** [description of last action]
**Next action:** [what should happen next]

## Context Snapshot

### What was being worked on

[Description of the task or objective]

### Progress so far

- [x] [Completed step 1]
- [x] [Completed step 2]
- [ ] [Next step - where we paused]
- [ ] [Remaining step]

### Key files touched

- `path/to/file1.py` — [what was changed]
- `path/to/file2.ts` — [what was changed]

### Uncommitted changes

[List any uncommitted work, or "None - all committed"]

## Open Questions

[Any questions that arose during work]

## Resume Instructions

1. Read this handoff
2. Check `git status` for any uncommitted work
3. Review the plan at [path]
4. Continue from "[next action]"

---
*Handoff by: [agent or user]*

</template>

<guidelines>

**Purpose:**
- Enable pause mid-work without losing context
- Support concurrent unrelated tasks
- Quick resume in new session

**Naming:**
- Date prefix for sorting: `2026-01-09-auth-implementation.md`
- Descriptive task name
- Lives in `context/session_handoffs/`

**Current State:**
- Where exactly we stopped
- What was last completed
- What should happen next

**Context Snapshot:**
- Enough detail to resume without re-reading everything
- Progress checklist
- Files touched
- Uncommitted changes warning

**Open Questions:**
- Things that came up during work
- May need user input on resume

**Resume Instructions:**
- Step-by-step to get back to productive state
- Reference to original plan

**Lifecycle:**
- Created on pause (/iscape:pause-work)
- Read on resume (/iscape:resume-work)
- Deleted after successful resume

</guidelines>
