# Custom GSD Design for Infrascape

**Date**: 2026-01-09
**Status**: Approved
**Author**: Mathew + Claude

## Overview

Customization of Get Shit Done (GSD) framework to integrate with Infrascape's existing:
- Agent framework (11 specialist agents with personas, roles, KPIs)
- Context directory structure (PRD, decision register, session handoffs)
- Organizational standards

## Design Goals

1. Phases live within existing `context/` directory structure
2. Tasks executed by subagents matching pre-defined agent personas
3. Project Lead agent orchestrates specialist agents
4. PRD remains authoritative, lightweight index for fast loading
5. Support concurrent unrelated tasks
6. Audit trail for deferred considerations

---

## Directory Structure

```
context/
├── prd/                          # Requirements (existing, authoritative)
├── decision_register/            # All decisions + deferred considerations
├── session_handoffs/             # Active work handoffs (cross-milestone)
├── active_work.md                # Dashboard of in-flight tasks
├── traceability.md               # Requirement → phase mapping
├── PROJECT.md                    # Lightweight index to PRD
│
└── milestones/
    └── v0.1/
        ├── MILESTONE.md          # Goals, success criteria, scope
        ├── decisions/            # Milestone-specific decisions
        └── phases/
            └── 01-poc/
                ├── PLAN.md       # Tasks with agent consultations
                ├── CONTEXT.md    # Discovery, research findings
                ├── SUMMARY.md    # Outcomes, what happened
                └── artifacts/    # Specs, diagrams, outputs
```

### Key Principles

- `context/` is the single source of truth for all non-code project information
- Milestones are self-contained units (v0.1, v0.2, etc.)
- Phases within milestones are flexible and can run concurrently
- Session handoffs live at project level (cross-milestone)

---

## Agent Model

### New Agent: Project Lead

**Archetype**: Tech Lead

**Mission**: Coordinate specialist agents to deliver phase objectives, balancing technical depth with delivery focus while maintaining quality and capturing deferred considerations.

**Interfaces**:
- **Consumes from**: product_strategy_agent, technical_architecture_agent
- **Provides to**: All implementation agents
- **Reviews**: All implementation agent outputs

**Key Behaviors**:
- Parses tasks from PLAN.md
- Identifies required consultations from natural language descriptions
- Spawns consultations to specialist agents
- Synthesizes inputs and delegates to appropriate specialist
- Reviews all outputs before proceeding
- Records deferred considerations in decision_register

### Execution Flow

```
PLAN.md loaded
    ↓
Project Lead parses tasks
    ↓
For each task:
    → Lead interprets which agents to consult (from task description)
    → Spawns consultations as needed
    → Synthesizes inputs
    → Delegates to appropriate specialist agent
    → Reviews output
    → Records any deferred considerations
    → Proceeds or escalates
    ↓
SUMMARY.md written
```

### Consultation Model

- Plans declare consultations in natural language within task descriptions
- Project Lead can add consultations dynamically based on judgment
- All implementation agent outputs reviewed by Lead before proceeding

### Existing Agents (unchanged)

| Agent | Role |
|-------|------|
| product_strategy_agent | Product vision, requirements prioritization |
| technical_architecture_agent | System design, API patterns, architecture decisions |
| implementation_engineering_agent | Backend services, API implementation |
| testing_qa_agent | Test strategy, test implementation |
| devops_infrastructure_agent | Deployment, infrastructure, CI/CD |
| security_compliance_agent | Security review, compliance requirements |
| ui_ux_design_agent | User experience, interface design |
| documentation_content_agent | Documentation, content |
| data_ingestion_agent | Data pipelines, ETL |
| rules_engine_agent | Business rules implementation |
| regulatory_intelligence_agent | Regulatory interpretation |

---

## Traceability & State Management

### PROJECT.md (lightweight index)

```markdown
# ComplianceFlow

Quick reference to PRD sections for context loading.

## Requirements Index
- Overview: @context/prd/01-overview.md
- Functional: @context/prd/05-functional-requirements.md
- Non-functional: @context/prd/06-non-functional-requirements.md
...

## Current State
- Active milestone: v0.1
- See: @context/active_work.md
```

### traceability.md

```markdown
# Requirements Traceability

| Requirement ID | PRD Section | Phase | Status |
|----------------|-------------|-------|--------|
| FR-001 | 05-functional-requirements.md#user-auth | v0.1/02-auth | Complete |
| FR-002 | 05-functional-requirements.md#data-import | v0.1/03-ingestion | In Progress |
| NFR-001 | 06-non-functional-requirements.md#security | v0.1/02-auth | Complete |
...
```

### active_work.md

```markdown
# Active Work

| Task | Milestone | Phase | Agent | Handoff |
|------|-----------|-------|-------|---------|
| Implement auth flow | v0.1 | 02-auth | implementation_engineering | 2026-01-09-auth-implementation.md |
| API contract review | v0.1 | 02-auth | technical_architecture | 2026-01-09-api-design.md |
```

### decision_register/ (extended format)

```markdown
## Decision: DEC-042
- Date: 2026-01-09
- Context: ...
- Decision: ...
- Status: Approved

## Deferred: DEF-012
- Date: 2026-01-09
- Raised by: implementation_engineering_agent
- Context: Suggested caching layer for API responses
- Disposition: Deferred to v0.2 - not in MVP scope
- Recorded by: project_lead_agent
```

---

## PLAN.md Format

Tasks use natural language to describe work and consultations:

```markdown
# Phase 02: Authentication - Plan 01

## Objective
Implement user authentication with role-based access control.

## Context
@context/prd/05-functional-requirements.md#user-auth
@context/prd/06-non-functional-requirements.md#security
@context/milestones/v0.1/phases/02-auth/CONTEXT.md

## Tasks

### Task 1: Define auth architecture
Design the authentication flow and token strategy. Consult with
security_compliance_agent on token storage and session handling
requirements. Technical_architecture_agent should validate the
approach fits our API patterns.

**Files**: context/milestones/v0.1/phases/02-auth/artifacts/auth-architecture.md
**Done when**: Architecture doc approved, security considerations documented

### Task 2: Implement auth endpoints
Build login, logout, and token refresh endpoints based on the
approved architecture. Testing_qa_agent should define test cases
before implementation begins.

**Files**: src/api/auth/*.py, tests/api/test_auth.py
**Done when**: Endpoints functional, tests passing

### Task 3: Integrate with frontend
Connect auth flow to UI. Consult ui_ux_design_agent on error
states and loading patterns.

**Files**: src/frontend/auth/*.tsx
**Done when**: Login flow works end-to-end

## Success Criteria
- Users can register, login, logout
- Tokens refresh correctly
- Role-based access enforced
```

---

## Adapted Commands (iscape namespace)

| Command | Behavior |
|---------|----------|
| `/iscape:new-project` | Creates `PROJECT.md` as index to existing PRD, initializes `traceability.md`, `active_work.md` |
| `/iscape:create-roadmap` | Creates milestone folders under `context/milestones/`, generates `MILESTONE.md` for each |
| `/iscape:plan-phase` | Creates `PLAN.md` in `context/milestones/vX/phases/XX-name/`, uses natural language task format |
| `/iscape:execute-plan` | Project Lead agent parses plan, orchestrates specialist agents, writes to `SUMMARY.md` and `decision_register/` |
| `/iscape:progress` | Reads `active_work.md`, shows in-flight tasks across milestones |
| `/iscape:pause-work` | Creates handoff in `session_handoffs/`, updates `active_work.md` |
| `/iscape:resume-work` | Reads from `session_handoffs/`, restores context |

### New/Modified Behaviors

- All execution flows through Project Lead agent
- Consultations parsed from natural language in tasks
- Deferred considerations recorded in `decision_register/`
- `traceability.md` updated when phases complete
- Concurrent tasks supported via `active_work.md`

---

## Project Lead Agent Definition

New files at `agents/infrascape-core/context/org/agent-framework/roles/project_lead_agent/`:

### persona.yaml

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

### role.yaml

```yaml
project_lead_agent:
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

  title: "Project Lead Agent"
  archetype: "Tech Lead"
  personality_inspirations:
    - Staff Engineers at Stripe
    - Tech Leads at Shopify
    - Engineering Managers at Atlassian

  mission: >
    Coordinate specialist agents to deliver phase objectives,
    balancing technical depth with delivery focus.

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

---

## Implementation Plan

### Phase 1: Foundation
1. Create Project Lead agent definition files
2. Create directory structure templates
3. Create file templates (PROJECT.md, traceability.md, active_work.md, MILESTONE.md)

### Phase 2: Namespace Rebrand
4. Copy `commands/gsd/` to `commands/iscape/`
5. Rename all command references from `gsd:` to `iscape:`
6. Update plugin.json to register `iscape` namespace
7. Update internal cross-references between commands

### Phase 3: Command Adaptation
8. Modify commands for new directory structure
9. Update path references (`.planning/` → `context/milestones/`)
10. Implement Project Lead orchestration in execute-plan
11. Add agent consultation parsing to plan execution

### Phase 4: Migration
12. Migrate existing `context/poc/` to `context/milestones/v0.1/phases/01-poc/`
13. Create PROJECT.md index for ComplianceFlow PRD
14. Initialize traceability.md with existing requirements

### Phase 5: Validation
15. Test full workflow with a new phase
16. Validate agent orchestration works as designed
17. Refine based on friction points

---

## Open Questions

None at this time. Design approved for implementation.
