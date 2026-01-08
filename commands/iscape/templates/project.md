# PROJECT.md Template (Index Style)

Template for `context/PROJECT.md` — lightweight index to PRD with project state.

<template>

# [Project Name]

## Quick Reference

**Core Value:** [One sentence - the ONE thing that matters most]

**Current Milestone:** [vX.X]

**Active Work:** See @context/active_work.md

## PRD Index

| Section | Path | Description |
|---------|------|-------------|
| Overview | @context/prd/01-overview.md | Problem, goals, success metrics |
| Background | @context/prd/02-background-context.md | Regulatory landscape, pain points |
| Users | @context/prd/03-users-personas.md | Primary users, jobs-to-be-done |
| Journeys | @context/prd/04-user-journeys.md | Key user flows |
| Functional | @context/prd/05-functional-requirements.md | Core features (Must/Should/Could) |
| Non-Functional | @context/prd/06-non-functional-requirements.md | Security, performance, compliance |
| Data | @context/prd/07-data-integrations.md | Data model, APIs, integrations |
| UX | @context/prd/08-ux-notes.md | Information architecture, wireframes |
| Reporting | @context/prd/09-reporting-auditability.md | Logs, evidence trails, exports |
| Risks | @context/prd/10-risks-assumptions.md | Risk register, assumptions |
| Milestones | @context/prd/11-milestones-rollout.md | Phased delivery plan |
| Questions | @context/prd/12-open-questions.md | Blockers, decisions needed |

## Key References

- **Traceability:** @context/traceability.md
- **Active Work:** @context/active_work.md
- **Decisions:** @context/decision_register/
- **Agent Framework:** @agents/infrascape-core/context/org/agent-framework/

---
*Last updated: [date] after [trigger]*

</template>

<guidelines>

**Purpose:**
- Lightweight entry point to project context
- Index to PRD sections for fast reference loading
- Does NOT duplicate PRD content
- PRD remains the authoritative source

**Quick Reference:**
- Core Value from PRD overview
- Current milestone being worked
- Pointer to active work dashboard

**PRD Index:**
- Map all PRD sections with @ references
- Brief description of each section's purpose
- Enables Claude to load specific sections as needed

**Key References:**
- Pointers to other context documents
- Agent framework location for specialist agents

**Maintenance:**
- Update "Last updated" when PRD sections change
- Update current milestone when transitioning
- Keep this file under 50 lines

</guidelines>
