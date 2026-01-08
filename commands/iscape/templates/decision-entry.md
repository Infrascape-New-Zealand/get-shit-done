# Decision Register Entry Template

Template for entries in `context/decision_register/` — decisions and deferrals.

<template>

## Decisions

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

## Deferrals

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
