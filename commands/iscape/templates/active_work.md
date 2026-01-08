# Active Work Template

Template for `context/active_work.md` — dashboard of in-flight tasks.

<template>

# Active Work

Dashboard of all in-flight tasks across milestones.

## Currently Active

| Task | Milestone | Phase | Lead Agent | Started | Handoff |
|------|-----------|-------|------------|---------|---------|
| [Task description] | v0.1 | 01-poc | project_lead | 2026-01-09 | [filename.md] |

## Recently Completed

| Task | Milestone | Phase | Completed | Summary |
|------|-----------|-------|-----------|---------|
| [Task description] | v0.1 | 01-poc | 2026-01-09 | @path/to/SUMMARY.md |

## Blocked

| Task | Milestone | Phase | Blocked Since | Blocker |
|------|-----------|-------|---------------|---------|
| - | - | - | - | - |

---
*Last updated: [date]*

</template>

<guidelines>

**Purpose:**
- Quick view of all work in progress
- Supports concurrent unrelated tasks
- Links to handoff files for context restoration

**Currently Active:**
- Tasks being worked on now
- Which milestone and phase
- Lead agent responsible
- Link to handoff file if paused

**Recently Completed:**
- Last 5 completed tasks
- Link to SUMMARY.md for details
- Older items archived

**Blocked:**
- Tasks that cannot proceed
- Blocker description
- Cleared when resolved

**Maintenance:**
- Add entry when task starts
- Update when task pauses (add handoff link)
- Move to Recently Completed when done
- Keep Recently Completed to last 5 items

</guidelines>
