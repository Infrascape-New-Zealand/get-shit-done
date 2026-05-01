---
name: iscape-review
description: "quality gates | code review debug audit security eval ui"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

Route to the appropriate quality / review skill based on the user's intent.
`iscape-code-review-fix` was absorbed by `iscape-code-review --fix` in #2790.

| User wants | Invoke |
|---|---|
| Review code for quality and correctness | iscape-code-review |
| Auto-fix code review findings | iscape-code-review --fix |
| Audit UAT / acceptance testing | iscape-audit-uat |
| Security review of a phase | iscape-secure-phase |
| Evaluate AI response quality | iscape-eval-review |
| Review UI for design and accessibility | iscape-ui-review |
| Validate phase outputs | iscape-validate-phase |
| Debug a failing feature or error | iscape-debug |
| Forensic investigation of a broken system | iscape-forensics |

Invoke the matched skill directly using the Skill tool.
