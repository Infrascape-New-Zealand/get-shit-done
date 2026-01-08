# Phase Plan Template

Template for `context/milestones/vX.X/phases/XX-name/PLAN.md` — executable task plan.

<template>

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
