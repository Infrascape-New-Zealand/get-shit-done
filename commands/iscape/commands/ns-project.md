---
name: iscape-project
description: "project lifecycle | milestones audits summary"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

Route to the appropriate project / milestone skill based on the user's intent.
`iscape-plan-milestone-gaps` was deleted by #2790 — gap planning now happens
inline as part of `iscape-audit-milestone`'s output.

| User wants | Invoke |
|---|---|
| Start a new project | iscape-new-project |
| Create a new milestone | iscape-new-milestone |
| Complete the current milestone | iscape-complete-milestone |
| Audit a milestone for issues | iscape-audit-milestone |
| Summarize milestone status | iscape-milestone-summary |

Invoke the matched skill directly using the Skill tool.
