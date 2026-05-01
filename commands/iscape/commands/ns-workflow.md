---
name: iscape-workflow
description: "workflow | discuss plan execute verify phase progress"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

Route to the appropriate phase-pipeline skill based on the user's intent.
Sub-skill names below are post-#2790 consolidated targets — `iscape-phase`
absorbs the former add/insert/remove/edit-phase commands and `iscape-progress`
absorbs the former next/do commands.

| User wants | Invoke |
|---|---|
| Gather context before planning | iscape-discuss-phase |
| Clarify what a phase delivers | iscape-spec-phase |
| Create a PLAN.md | iscape-plan-phase |
| Execute plans in a phase | iscape-execute-phase |
| Verify built features through UAT | iscape-verify-work |
| Add / insert / remove / edit a phase | iscape-phase |
| Advance to the next logical step | iscape-progress |
| Offload planning to the ultraplan cloud | iscape-ultraplan-phase |
| Cross-AI plan review convergence loop | iscape-plan-review-convergence |

Invoke the matched skill directly using the Skill tool.
