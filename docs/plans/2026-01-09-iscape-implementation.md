# Iscape Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Transform GSD into Iscape - a customized implementation framework for Infrascape with Project Lead orchestration, milestone-based organization, and specialist agent integration.

**Architecture:** Fork GSD commands to `iscape:` namespace, replace `.planning/` structure with `context/milestones/` hierarchy, add Project Lead agent as orchestrator that delegates to specialist agents.

**Tech Stack:** Markdown commands, YAML agent definitions, Claude Code plugin system

---

## Phase 1: Foundation

Create the Project Lead agent definition and new file templates.

### Task 1.1: Create Project Lead agent directory

**Files:**
- Create: `commands/iscape/agents/project_lead_agent/persona.yaml`

**Step 1: Create directory structure**

```bash
mkdir -p commands/iscape/agents/project_lead_agent
```

**Step 2: Create persona.yaml**

```yaml
project_lead_agent_persona:
  name: "Project Lead Agent"
  archetype: "Tech Lead"
  mission: >
    Coordinate specialist agents to deliver phase objectives,
    balancing technical depth with delivery focus while maintaining
    quality and capturing deferred considerations.

  personality_traits:
    - Technically credible but delivery-focused
    - Synthesizes diverse inputs into clear direction
    - Protective of scope without being rigid
    - Proactive about surfacing and recording risks
    - Balances speed with quality

  strengths:
    coordination:
      - Breaking down complex work into agent-appropriate tasks
      - Identifying when specialist consultation is needed
      - Synthesizing conflicting advice into decisions
    technical:
      - Deep enough to validate implementation approaches
      - Recognizes architectural implications
      - Understands testing and quality trade-offs
    delivery:
      - Keeps work moving, unblocks quickly
      - Manages scope creep constructively
      - Documents what's deferred, not ignored

  limitations:
    - Does not replace specialist expertise
    - Does not make product strategy decisions alone
    - Does not implement code directly
    - Does not override security_compliance_agent on security matters

  communication_style:
    tone: "clear, decisive, collaborative"
    avoids:
      - Analysis paralysis
      - Unilateral decisions on specialist domains
      - Ignoring raised concerns (always records)

  collaboration_contract:
    consumes_from:
      - product_strategy_agent
      - technical_architecture_agent
      - all implementation agents (for review)
    provides_to:
      - implementation_engineering_agent
      - testing_qa_agent
      - devops_infrastructure_agent
      - ui_ux_design_agent
      - documentation_content_agent

  success_definition: >
    Phases complete on objective with quality maintained,
    all concerns addressed or explicitly deferred with audit trail.
```

**Step 3: Commit**

```bash
git add commands/iscape/agents/project_lead_agent/persona.yaml
git commit -m "feat(iscape): add Project Lead agent persona"
```

---

### Task 1.2: Create Project Lead role.yaml

**Files:**
- Create: `commands/iscape/agents/project_lead_agent/role.yaml`

**Step 1: Create role.yaml**

```yaml
project_lead_agent:
  title: "Project Lead Agent"
  archetype: "Tech Lead"

  personality_inspirations:
    - Staff Engineers at Stripe
    - Tech Leads at Shopify
    - Engineering Managers at Atlassian

  mission: >
    Coordinate specialist agents to deliver phase objectives,
    balancing technical depth with delivery focus.

  agent_context:
    objective: "<current phase or milestone objective>"
    scope:
      level: "milestone"
      product_or_initiative: "<milestone name>"
    timeline:
      horizon: "<milestone target>"
      cadence: "per-task updates"
    success_criteria:
      - "Phase objectives met"
      - "All concerns addressed or recorded"
      - "Quality maintained"
    artifacts:
      inputs_root: "context/milestones/<version>/phases/<phase>/"
      outputs_root: "context/milestones/<version>/phases/<phase>/"
      naming_convention: "PLAN.md, CONTEXT.md, SUMMARY.md"
    tools_and_access:
      - "All project repositories"
      - "PRD and context documents"
      - "Decision register"
    collaboration:
      handoff_format: "SUMMARY.md + decision_register updates"
      escalation_path: "User/stakeholder"
      status_reporting: "active_work.md"
    references:
      architecture_standards: "context/org/architecture-standards/"
      brand_guidelines: "context/org/brand/"
      governance_rules: "context/org/governance/"
      operating_guidelines: "context/org/guidelines/"
      operations_playbooks: "context/org/operations/"

  responsibilities:
    - Parse PLAN.md and identify required consultations
    - Orchestrate specialist agent consultations
    - Synthesize inputs into actionable direction
    - Delegate tasks to appropriate specialist agents
    - Review all implementation outputs
    - Record deferred considerations in decision_register
    - Update traceability.md on phase completion
    - Maintain active_work.md for in-flight tasks

  inputs:
    - PLAN.md
    - CONTEXT.md
    - PRD sections (via PROJECT.md index)
    - Specialist agent outputs

  outputs:
    - SUMMARY.md
    - decision_register updates (decisions + deferrals)
    - traceability.md updates
    - active_work.md updates

  interfaces:
    consumes_from:
      - product_strategy_agent
      - technical_architecture_agent
      - all implementation agents (review)
    provides_to:
      - implementation_engineering_agent
      - testing_qa_agent
      - devops_infrastructure_agent
      - ui_ux_design_agent
      - documentation_content_agent

  boundaries:
    - "Never override security_compliance_agent on security matters"
    - "Never make product strategy decisions without product_strategy_agent"
    - "Always record deferred concerns - never ignore"
    - "Review all implementation outputs before proceeding"
```

**Step 2: Commit**

```bash
git add commands/iscape/agents/project_lead_agent/role.yaml
git commit -m "feat(iscape): add Project Lead agent role definition"
```

---

### Task 1.3: Create PROJECT.md template (index style)

**Files:**
- Create: `commands/iscape/templates/project.md`

**Step 1: Create templates directory**

```bash
mkdir -p commands/iscape/templates
```

**Step 2: Create project.md template**

```markdown
# PROJECT.md Template (Index Style)

Template for `context/PROJECT.md` — lightweight index to PRD with project state.

<template>

```markdown
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
```

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
```

**Step 3: Commit**

```bash
git add commands/iscape/templates/project.md
git commit -m "feat(iscape): add PROJECT.md index template"
```

---

### Task 1.4: Create traceability.md template

**Files:**
- Create: `commands/iscape/templates/traceability.md`

**Step 1: Create traceability.md template**

```markdown
# Traceability Template

Template for `context/traceability.md` — requirements to phase mapping.

<template>

```markdown
# Requirements Traceability Matrix

Maps PRD requirements to implementation phases.

## Functional Requirements

| ID | Requirement | PRD Section | Milestone | Phase | Status |
|----|-------------|-------------|-----------|-------|--------|
| FR-001 | [Requirement name] | 05-functional#section | v0.1 | 01-poc | Not Started |
| FR-002 | [Requirement name] | 05-functional#section | v0.1 | 02-name | In Progress |
| FR-003 | [Requirement name] | 05-functional#section | v0.1 | 02-name | Complete |

## Non-Functional Requirements

| ID | Requirement | PRD Section | Milestone | Phase | Status |
|----|-------------|-------------|-----------|-------|--------|
| NFR-001 | [Requirement name] | 06-non-functional#section | v0.1 | 01-poc | Not Started |

## Coverage Summary

| Milestone | Total Reqs | Not Started | In Progress | Complete | Coverage |
|-----------|------------|-------------|-------------|----------|----------|
| v0.1 | 0 | 0 | 0 | 0 | 0% |

---
*Last updated: [date]*
```

</template>

<guidelines>

**Purpose:**
- Single source of truth for requirement coverage
- Links PRD requirements to implementation
- Tracks status across milestones

**Columns:**
- ID: Unique identifier (FR-XXX, NFR-XXX)
- Requirement: Brief name/description
- PRD Section: Reference to PRD file and section
- Milestone: Which milestone addresses this
- Phase: Specific phase within milestone
- Status: Not Started / In Progress / Complete

**Coverage Summary:**
- Aggregate view per milestone
- Calculate coverage percentage
- Updated when phases complete

**Maintenance:**
- Update status when phase completes
- Add new requirements as PRD evolves
- Mark requirements deferred if moved to future milestone

</guidelines>
```

**Step 2: Commit**

```bash
git add commands/iscape/templates/traceability.md
git commit -m "feat(iscape): add traceability matrix template"
```

---

### Task 1.5: Create active_work.md template

**Files:**
- Create: `commands/iscape/templates/active_work.md`

**Step 1: Create active_work.md template**

```markdown
# Active Work Template

Template for `context/active_work.md` — dashboard of in-flight tasks.

<template>

```markdown
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
```

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
```

**Step 2: Commit**

```bash
git add commands/iscape/templates/active_work.md
git commit -m "feat(iscape): add active_work dashboard template"
```

---

### Task 1.6: Create MILESTONE.md template

**Files:**
- Create: `commands/iscape/templates/milestone.md`

**Step 1: Create milestone.md template**

```markdown
# Milestone Template

Template for `context/milestones/vX.X/MILESTONE.md` — milestone scope and goals.

<template>

```markdown
# Milestone [vX.X]: [Name]

## Objective

[2-3 sentences describing what this milestone delivers and why it matters]

## Success Criteria

- [ ] [Measurable outcome 1]
- [ ] [Measurable outcome 2]
- [ ] [Measurable outcome 3]

## Scope

### In Scope

- [Feature/capability 1]
- [Feature/capability 2]
- [Feature/capability 3]

### Out of Scope

- [Exclusion 1] — deferred to [vX.Y]
- [Exclusion 2] — not required for this milestone

## Phases

| Phase | Name | Objective | Status |
|-------|------|-----------|--------|
| 01 | [name] | [brief objective] | Not Started |
| 02 | [name] | [brief objective] | Not Started |

## Dependencies

- [Dependency 1]: [status]
- [Dependency 2]: [status]

## Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| [Risk description] | High/Med/Low | [Mitigation strategy] |

---
*Created: [date]*
*Target: [date]*
```

</template>

<guidelines>

**Purpose:**
- Self-contained milestone definition
- Clear scope boundaries
- Phase overview without detail

**Objective:**
- What this milestone delivers
- Why it matters (business value)
- 2-3 sentences max

**Success Criteria:**
- Measurable outcomes
- Checkboxes for tracking
- Updated when phases complete

**Scope:**
- In Scope: What's included
- Out of Scope: What's excluded with reason
- Prevents scope creep

**Phases:**
- Overview table only
- Detailed plans live in phase folders
- Status: Not Started / In Progress / Complete

**Dependencies:**
- External dependencies
- Cross-milestone dependencies
- Track status

**Risks:**
- Milestone-specific risks
- Impact assessment
- Mitigation strategies

</guidelines>
```

**Step 2: Commit**

```bash
git add commands/iscape/templates/milestone.md
git commit -m "feat(iscape): add MILESTONE.md template"
```

---

### Task 1.7: Create phase PLAN.md template

**Files:**
- Create: `commands/iscape/templates/phase-plan.md`

**Step 1: Create phase-plan.md template**

```markdown
# Phase Plan Template

Template for `context/milestones/vX.X/phases/XX-name/PLAN.md` — executable task plan.

<template>

```markdown
# Phase [XX]: [Name] — Plan [YY]

## Objective

[What this plan accomplishes and why]

## Context

@context/PROJECT.md
@context/prd/[relevant-section].md
@context/milestones/[version]/phases/[phase]/CONTEXT.md

## Tasks

### Task 1: [Task name]

[Description of what needs to be done. Include consultation requirements in natural language.]

Consult [agent_name] on [specific aspect]. [Another_agent] should validate [specific concern].

**Files:** [paths to files that will be created or modified]

**Done when:** [specific, verifiable completion criteria]

### Task 2: [Task name]

[Description with consultation requirements as needed]

**Files:** [paths]

**Done when:** [criteria]

### Task 3: [Task name]

[Description]

**Files:** [paths]

**Done when:** [criteria]

## Success Criteria

- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

## Verification

[How to verify the plan succeeded — commands to run, behaviors to test]
```

</template>

<guidelines>

**Purpose:**
- Executable plan for Project Lead agent
- Natural language task descriptions
- Embedded consultation requirements

**Objective:**
- What this specific plan accomplishes
- Connect to larger phase/milestone goals
- One sentence

**Context:**
- @ references to relevant documents
- PROJECT.md for overall context
- Relevant PRD sections
- Phase CONTEXT.md if exists

**Tasks:**
- 2-3 tasks per plan (keeps scope small)
- Natural language descriptions
- Consultation requirements inline
- Specific files affected
- Clear "Done when" criteria

**Consultation Format:**
- "Consult [agent] on [topic]"
- "Have [agent] review [artifact]"
- "[Agent] should validate [concern]"
- Project Lead interprets and orchestrates

**Success Criteria:**
- Measurable outcomes
- Checkboxes for tracking
- Updated by Project Lead in SUMMARY.md

**Verification:**
- How to prove plan succeeded
- Commands to run
- Behaviors to observe

</guidelines>

<project_lead_interpretation>

When Project Lead reads this plan:

1. Parse tasks for consultation keywords
2. Identify agents to consult
3. Determine consultation order
4. For each task:
   - Gather required consultations
   - Synthesize inputs
   - Delegate to appropriate specialist
   - Review output
   - Record any deferred considerations
5. Write SUMMARY.md with outcomes

</project_lead_interpretation>
```

**Step 2: Commit**

```bash
git add commands/iscape/templates/phase-plan.md
git commit -m "feat(iscape): add phase PLAN.md template with consultation format"
```

---

### Task 1.8: Create decision register entry template

**Files:**
- Create: `commands/iscape/templates/decision-entry.md`

**Step 1: Create decision-entry.md template**

```markdown
# Decision Register Entry Template

Template for entries in `context/decision_register/` — decisions and deferrals.

<template>

## Decisions

```markdown
## Decision: DEC-[XXX]

**Date:** [YYYY-MM-DD]
**Phase:** [milestone/phase reference]
**Made by:** [agent or user]

### Context

[What situation or question prompted this decision]

### Options Considered

1. [Option A] — [pros/cons]
2. [Option B] — [pros/cons]
3. [Option C] — [pros/cons]

### Decision

[What was decided and why]

### Outcome

[Status: Pending / Implemented / Validated / Revisit]
[Notes on how it's working if known]
```

## Deferrals

```markdown
## Deferred: DEF-[XXX]

**Date:** [YYYY-MM-DD]
**Phase:** [milestone/phase reference]
**Raised by:** [agent_name]
**Recorded by:** project_lead_agent

### Context

[What was raised and why it matters]

### Disposition

[Why it was deferred — not in scope, future milestone, needs more info]

### Target

[When to revisit: vX.Y, specific phase, or "backlog"]
```

</template>

<guidelines>

**Decisions:**
- Significant choices affecting future work
- Record options considered
- Include rationale
- Track outcome over time

**Deferrals:**
- Concerns raised but not acted on
- Must record who raised it
- Must record why deferred
- Must record when to revisit
- Creates audit trail

**Numbering:**
- DEC-001, DEC-002 for decisions
- DEF-001, DEF-002 for deferrals
- Sequential within register

**File Organization:**
- One file per milestone: `context/decision_register/v0.1-decisions.md`
- Or one file per major decision area
- Keep related decisions together

</guidelines>
```

**Step 2: Commit**

```bash
git add commands/iscape/templates/decision-entry.md
git commit -m "feat(iscape): add decision register entry template with deferrals"
```

---

### Task 1.9: Create phase SUMMARY.md template

**Files:**
- Create: `commands/iscape/templates/phase-summary.md`

**Step 1: Create phase-summary.md template**

```markdown
# Phase Summary Template

Template for `context/milestones/vX.X/phases/XX-name/SUMMARY.md` — execution outcomes.

<template>

```markdown
# Phase [XX]: [Name] — Plan [YY] Summary

**Executed:** [YYYY-MM-DD]
**Duration:** [X minutes]
**Lead:** project_lead_agent

## Outcome

[Brief summary of what was accomplished]

## Tasks Completed

### Task 1: [Name]

**Status:** Complete / Partial / Blocked
**Commits:** [hash1], [hash2]

[What was done, any deviations from plan]

**Consultations:**
- [agent_name]: [what they contributed]
- [agent_name]: [what they contributed]

### Task 2: [Name]

**Status:** [status]
**Commits:** [hashes]

[Details]

## Decisions Made

| Decision | Rationale | Logged |
|----------|-----------|--------|
| [Choice] | [Why] | DEC-XXX |

## Deferred Considerations

| Consideration | Raised By | Disposition | Logged |
|---------------|-----------|-------------|--------|
| [Concern] | [agent] | [Why deferred] | DEF-XXX |

## Success Criteria

- [x] [Criterion 1]
- [x] [Criterion 2]
- [ ] [Criterion 3 - not met, reason]

## Next Steps

[What should happen next based on this work]

---
*Generated by project_lead_agent*
```

</template>

<guidelines>

**Purpose:**
- Record of what actually happened
- Links to commits for traceability
- Captures decisions and deferrals
- Enables future context restoration

**Outcome:**
- Brief summary (2-3 sentences)
- What was accomplished vs planned

**Tasks Completed:**
- Status of each task
- Commit hashes for code changes
- Deviations from plan
- Consultations that occurred

**Decisions Made:**
- Link to decision register entry
- Brief rationale inline

**Deferred Considerations:**
- What was raised but not acted on
- Who raised it
- Why deferred
- Link to deferral entry in register

**Success Criteria:**
- Checkboxes from plan
- Mark met/unmet
- Explain any failures

**Next Steps:**
- What logically follows
- May inform next plan

</guidelines>
```

**Step 2: Commit**

```bash
git add commands/iscape/templates/phase-summary.md
git commit -m "feat(iscape): add phase SUMMARY.md template"
```

---

### Task 1.10: Create session handoff template

**Files:**
- Create: `commands/iscape/templates/session-handoff.md`

**Step 1: Create session-handoff.md template**

```markdown
# Session Handoff Template

Template for `context/session_handoffs/YYYY-MM-DD-[task-name].md` — pause/resume context.

<template>

```markdown
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
```

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
```

**Step 2: Commit**

```bash
git add commands/iscape/templates/session-handoff.md
git commit -m "feat(iscape): add session handoff template"
```

---

## Phase 2: Namespace Rebrand

Copy GSD commands to iscape namespace and rename all references.

### Task 2.1: Copy command directory

**Files:**
- Create: `commands/iscape/` (copy from `commands/gsd/`)

**Step 1: Copy entire gsd directory to iscape**

```bash
cp -r commands/gsd commands/iscape
```

**Step 2: Verify copy**

```bash
ls commands/iscape/
```

Expected: All .md files from gsd directory

**Step 3: Commit**

```bash
git add commands/iscape/*.md
git commit -m "chore(iscape): copy gsd commands as starting point"
```

---

### Task 2.2: Rename command frontmatter (batch 1)

**Files:**
- Modify: `commands/iscape/new-project.md`
- Modify: `commands/iscape/create-roadmap.md`
- Modify: `commands/iscape/map-codebase.md`
- Modify: `commands/iscape/plan-phase.md`
- Modify: `commands/iscape/execute-plan.md`

**Step 1: Update frontmatter in each file**

Change `name: gsd:*` to `name: iscape:*` in the YAML frontmatter of each file.

Example for new-project.md:
```yaml
---
name: iscape:new-project
description: Initialize a new project with deep context gathering and PROJECT.md
---
```

**Step 2: Update internal references**

Search and replace within each file:
- `/gsd:` → `/iscape:`
- `gsd:` → `iscape:` (in description text)

**Step 3: Commit**

```bash
git add commands/iscape/new-project.md commands/iscape/create-roadmap.md commands/iscape/map-codebase.md commands/iscape/plan-phase.md commands/iscape/execute-plan.md
git commit -m "refactor(iscape): rename commands batch 1 (project, roadmap, map, plan, execute)"
```

---

### Task 2.3: Rename command frontmatter (batch 2)

**Files:**
- Modify: `commands/iscape/progress.md`
- Modify: `commands/iscape/complete-milestone.md`
- Modify: `commands/iscape/discuss-milestone.md`
- Modify: `commands/iscape/new-milestone.md`
- Modify: `commands/iscape/add-phase.md`

**Step 1: Update frontmatter in each file**

Change `name: gsd:*` to `name: iscape:*`

**Step 2: Update internal references**

Search and replace `/gsd:` → `/iscape:` within each file.

**Step 3: Commit**

```bash
git add commands/iscape/progress.md commands/iscape/complete-milestone.md commands/iscape/discuss-milestone.md commands/iscape/new-milestone.md commands/iscape/add-phase.md
git commit -m "refactor(iscape): rename commands batch 2 (progress, milestone commands)"
```

---

### Task 2.4: Rename command frontmatter (batch 3)

**Files:**
- Modify: `commands/iscape/insert-phase.md`
- Modify: `commands/iscape/remove-phase.md`
- Modify: `commands/iscape/discuss-phase.md`
- Modify: `commands/iscape/research-phase.md`
- Modify: `commands/iscape/list-phase-assumptions.md`

**Step 1: Update frontmatter in each file**

Change `name: gsd:*` to `name: iscape:*`

**Step 2: Update internal references**

Search and replace `/gsd:` → `/iscape:` within each file.

**Step 3: Commit**

```bash
git add commands/iscape/insert-phase.md commands/iscape/remove-phase.md commands/iscape/discuss-phase.md commands/iscape/research-phase.md commands/iscape/list-phase-assumptions.md
git commit -m "refactor(iscape): rename commands batch 3 (phase management)"
```

---

### Task 2.5: Rename command frontmatter (batch 4)

**Files:**
- Modify: `commands/iscape/pause-work.md`
- Modify: `commands/iscape/resume-work.md`
- Modify: `commands/iscape/consider-issues.md`
- Modify: `commands/iscape/verify-work.md`
- Modify: `commands/iscape/plan-fix.md`
- Modify: `commands/iscape/help.md`

**Step 1: Update frontmatter in each file**

Change `name: gsd:*` to `name: iscape:*`

**Step 2: Update internal references**

Search and replace `/gsd:` → `/iscape:` within each file.

**Step 3: Commit**

```bash
git add commands/iscape/pause-work.md commands/iscape/resume-work.md commands/iscape/consider-issues.md commands/iscape/verify-work.md commands/iscape/plan-fix.md commands/iscape/help.md
git commit -m "refactor(iscape): rename commands batch 4 (work management, help)"
```

---

### Task 2.6: Create iscape plugin.json

**Files:**
- Create: `commands/iscape/plugin.json`

**Step 1: Create plugin.json for iscape**

```json
{
  "name": "iscape",
  "version": "1.0.0",
  "description": "Infrascape development framework - milestone-based planning with specialist agent orchestration",
  "author": {
    "name": "Infrascape"
  },
  "license": "MIT",
  "commands": [
    "./new-project.md",
    "./create-roadmap.md",
    "./map-codebase.md",
    "./plan-phase.md",
    "./execute-plan.md",
    "./progress.md",
    "./complete-milestone.md",
    "./discuss-milestone.md",
    "./new-milestone.md",
    "./add-phase.md",
    "./insert-phase.md",
    "./remove-phase.md",
    "./discuss-phase.md",
    "./research-phase.md",
    "./list-phase-assumptions.md",
    "./pause-work.md",
    "./resume-work.md",
    "./consider-issues.md",
    "./verify-work.md",
    "./plan-fix.md",
    "./help.md"
  ]
}
```

**Step 2: Commit**

```bash
git add commands/iscape/plugin.json
git commit -m "feat(iscape): add plugin.json for iscape namespace"
```

---

### Task 2.7: Update help.md with iscape branding

**Files:**
- Modify: `commands/iscape/help.md`

**Step 1: Update help content**

Replace all GSD references with Iscape:
- "Get Shit Done" → "Infrascape Development Framework"
- "GSD" → "Iscape"
- Update command examples to use `/iscape:*`

**Step 2: Update command list to reflect iscape namespace**

Ensure all command references use `iscape:` prefix.

**Step 3: Commit**

```bash
git add commands/iscape/help.md
git commit -m "docs(iscape): update help.md with iscape branding"
```

---

## Phase 3: Command Adaptation

Modify commands to use new directory structure and Project Lead orchestration.

### Task 3.1: Update path constants

**Files:**
- Modify: All commands in `commands/iscape/`

**Step 1: Create a reference document for path mapping**

Old GSD paths → New Iscape paths:
- `.planning/` → `context/`
- `.planning/PROJECT.md` → `context/PROJECT.md`
- `.planning/STATE.md` → `context/active_work.md` (partial)
- `.planning/ROADMAP.md` → `context/milestones/` structure
- `.planning/phases/XX-name/` → `context/milestones/vX.X/phases/XX-name/`
- `.planning/ISSUES.md` → `context/decision_register/` (deferrals)

**Step 2: Update new-project.md paths**

Change all `.planning/` references to `context/` structure.

**Step 3: Commit**

```bash
git add commands/iscape/new-project.md
git commit -m "refactor(iscape): update new-project paths to context/ structure"
```

---

### Task 3.2: Update create-roadmap.md for milestone structure

**Files:**
- Modify: `commands/iscape/create-roadmap.md`

**Step 1: Update to create milestone folders**

Instead of `.planning/phases/`, create:
- `context/milestones/v0.1/MILESTONE.md`
- `context/milestones/v0.1/phases/`
- `context/milestones/v0.1/decisions/`

**Step 2: Reference new templates**

Use `@commands/iscape/templates/milestone.md` template.

**Step 3: Initialize traceability.md and active_work.md**

Add steps to create these files during roadmap creation.

**Step 4: Commit**

```bash
git add commands/iscape/create-roadmap.md
git commit -m "refactor(iscape): update create-roadmap for milestone structure"
```

---

### Task 3.3: Update plan-phase.md for new structure

**Files:**
- Modify: `commands/iscape/plan-phase.md`

**Step 1: Update paths**

Plans created at `context/milestones/vX.X/phases/XX-name/PLAN.md`

**Step 2: Reference new plan template**

Use `@commands/iscape/templates/phase-plan.md` template.

**Step 3: Add consultation guidance**

Include instructions for writing tasks with agent consultation requirements.

**Step 4: Commit**

```bash
git add commands/iscape/plan-phase.md
git commit -m "refactor(iscape): update plan-phase for milestone structure and consultation format"
```

---

### Task 3.4: Update execute-plan.md for Project Lead orchestration

**Files:**
- Modify: `commands/iscape/execute-plan.md`

**Step 1: Add Project Lead agent loading**

Reference Project Lead persona and role:
```
@commands/iscape/agents/project_lead_agent/persona.yaml
@commands/iscape/agents/project_lead_agent/role.yaml
```

**Step 2: Update execution flow**

1. Project Lead parses PLAN.md
2. Identifies consultation requirements from task descriptions
3. Spawns consultations to specialist agents
4. Synthesizes inputs
5. Delegates to appropriate specialist
6. Reviews all outputs
7. Records deferrals in decision_register

**Step 3: Update output paths**

- SUMMARY.md at `context/milestones/vX.X/phases/XX-name/SUMMARY.md`
- Deferrals at `context/decision_register/vX.X-decisions.md`
- Update `context/active_work.md`

**Step 4: Reference new templates**

Use `@commands/iscape/templates/phase-summary.md` template.

**Step 5: Commit**

```bash
git add commands/iscape/execute-plan.md
git commit -m "refactor(iscape): update execute-plan for Project Lead orchestration"
```

---

### Task 3.5: Update progress.md for active_work.md

**Files:**
- Modify: `commands/iscape/progress.md`

**Step 1: Update to read active_work.md**

Instead of STATE.md, read `context/active_work.md` for current work.

**Step 2: Update milestone/phase discovery**

Scan `context/milestones/` for milestone status.

**Step 3: Update traceability reporting**

Include coverage summary from `context/traceability.md`.

**Step 4: Commit**

```bash
git add commands/iscape/progress.md
git commit -m "refactor(iscape): update progress for active_work and traceability"
```

---

### Task 3.6: Update pause-work.md for session handoffs

**Files:**
- Modify: `commands/iscape/pause-work.md`

**Step 1: Update handoff path**

Create handoffs at `context/session_handoffs/YYYY-MM-DD-[task].md`

**Step 2: Reference new template**

Use `@commands/iscape/templates/session-handoff.md` template.

**Step 3: Update active_work.md**

Add step to update `context/active_work.md` with handoff reference.

**Step 4: Commit**

```bash
git add commands/iscape/pause-work.md
git commit -m "refactor(iscape): update pause-work for session_handoffs structure"
```

---

### Task 3.7: Update resume-work.md for session handoffs

**Files:**
- Modify: `commands/iscape/resume-work.md`

**Step 1: Update to read from session_handoffs**

Read handoffs from `context/session_handoffs/`

**Step 2: Update active_work.md on resume**

Clear handoff reference when work resumes.

**Step 3: Commit**

```bash
git add commands/iscape/resume-work.md
git commit -m "refactor(iscape): update resume-work for session_handoffs structure"
```

---

### Task 3.8: Update remaining commands for new paths

**Files:**
- Modify: `commands/iscape/complete-milestone.md`
- Modify: `commands/iscape/new-milestone.md`
- Modify: `commands/iscape/add-phase.md`
- Modify: `commands/iscape/insert-phase.md`
- Modify: `commands/iscape/remove-phase.md`

**Step 1: Update all path references**

Change `.planning/` to `context/milestones/` structure.

**Step 2: Update milestone completion**

Archive milestone properly within `context/milestones/` structure.

**Step 3: Commit**

```bash
git add commands/iscape/complete-milestone.md commands/iscape/new-milestone.md commands/iscape/add-phase.md commands/iscape/insert-phase.md commands/iscape/remove-phase.md
git commit -m "refactor(iscape): update milestone/phase commands for new structure"
```

---

## Phase 4: Migration

Migrate existing ComplianceFlow POC to new structure.

### Task 4.1: Create milestone v0.1 structure

**Files:**
- Create: `[ComplianceFlow]/context/milestones/v0.1/MILESTONE.md`
- Create: `[ComplianceFlow]/context/milestones/v0.1/phases/`
- Create: `[ComplianceFlow]/context/milestones/v0.1/decisions/`

**Step 1: Create directory structure**

```bash
mkdir -p [ComplianceFlow]/context/milestones/v0.1/phases
mkdir -p [ComplianceFlow]/context/milestones/v0.1/decisions
```

**Step 2: Create MILESTONE.md**

Use the template from `commands/iscape/templates/milestone.md` to create the v0.1 milestone document.

**Step 3: Commit**

```bash
git add context/milestones/
git commit -m "feat: initialize v0.1 milestone structure"
```

---

### Task 4.2: Migrate POC to phase 01

**Files:**
- Move: `context/poc/` → `context/milestones/v0.1/phases/01-poc/`

**Step 1: Move POC directory**

```bash
mv context/poc context/milestones/v0.1/phases/01-poc
```

**Step 2: Create CONTEXT.md for POC phase**

Summarize what the POC contains.

**Step 3: Commit**

```bash
git add context/milestones/v0.1/phases/01-poc/
git commit -m "refactor: migrate POC to v0.1 phase 01"
```

---

### Task 4.3: Create PROJECT.md index

**Files:**
- Create: `[ComplianceFlow]/context/PROJECT.md`

**Step 1: Create PROJECT.md**

Use the template from `commands/iscape/templates/project.md` to create an index pointing to existing PRD sections.

**Step 2: Commit**

```bash
git add context/PROJECT.md
git commit -m "docs: create PROJECT.md index to PRD"
```

---

### Task 4.4: Initialize traceability.md

**Files:**
- Create: `[ComplianceFlow]/context/traceability.md`

**Step 1: Create traceability.md**

Use template and populate with initial requirements from PRD.

**Step 2: Commit**

```bash
git add context/traceability.md
git commit -m "docs: initialize traceability matrix"
```

---

### Task 4.5: Initialize active_work.md

**Files:**
- Create: `[ComplianceFlow]/context/active_work.md`

**Step 1: Create active_work.md**

Use template and initialize with empty state.

**Step 2: Commit**

```bash
git add context/active_work.md
git commit -m "docs: initialize active work dashboard"
```

---

## Phase 5: Validation

Test the full workflow with a real phase.

### Task 5.1: Test /iscape:progress

**Step 1: Run progress command**

```
/iscape:progress
```

**Step 2: Verify output**

- Shows current milestone (v0.1)
- Shows POC phase status
- Shows traceability coverage
- Shows active work (empty)

**Step 3: Document any issues**

Note any path errors or missing files.

---

### Task 5.2: Test /iscape:plan-phase

**Step 1: Plan a new phase**

```
/iscape:plan-phase 02-auth
```

**Step 2: Verify plan creation**

- PLAN.md created at correct path
- Uses natural language task format
- Includes consultation requirements

**Step 3: Verify phase folder structure**

- `context/milestones/v0.1/phases/02-auth/PLAN.md` exists
- `context/milestones/v0.1/phases/02-auth/artifacts/` created

---

### Task 5.3: Test /iscape:execute-plan

**Step 1: Execute the plan**

```
/iscape:execute-plan context/milestones/v0.1/phases/02-auth/PLAN.md
```

**Step 2: Verify Project Lead orchestration**

- Consultations identified from task descriptions
- Specialist agents consulted as needed
- Outputs reviewed by Project Lead
- Deferrals recorded if any

**Step 3: Verify outputs**

- SUMMARY.md created
- decision_register updated (if decisions/deferrals)
- active_work.md updated
- traceability.md updated

---

### Task 5.4: Test /iscape:pause-work and /iscape:resume-work

**Step 1: Start work on a task**

Begin executing a plan.

**Step 2: Pause mid-work**

```
/iscape:pause-work
```

**Step 3: Verify handoff**

- Handoff created in `context/session_handoffs/`
- active_work.md updated with handoff reference

**Step 4: Resume work**

```
/iscape:resume-work
```

**Step 5: Verify resume**

- Context restored from handoff
- Work continues from pause point
- Handoff cleared after successful resume

---

### Task 5.5: Document findings and refine

**Step 1: Document any issues found**

Create `docs/plans/2026-01-09-iscape-validation-findings.md`

**Step 2: Create follow-up tasks**

If issues found, plan fixes.

**Step 3: Commit findings**

```bash
git add docs/plans/2026-01-09-iscape-validation-findings.md
git commit -m "docs: iscape validation findings and refinements"
```

---

## Summary

This plan transforms GSD into Iscape through 5 phases:

1. **Foundation** (10 tasks): Agent definition, templates
2. **Namespace Rebrand** (7 tasks): gsd → iscape
3. **Command Adaptation** (8 tasks): New directory structure, Project Lead
4. **Migration** (5 tasks): ComplianceFlow POC migration
5. **Validation** (5 tasks): End-to-end testing

Total: 35 tasks

Each task is atomic with clear completion criteria and commits.
