---
name: iscape-context
description: "codebase intelligence | map graphify docs learnings"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

Route to the appropriate codebase-intelligence skill based on the user's intent.
`iscape-scan` and `iscape-intel` were folded into `iscape-map-codebase` flags by #2790.

| User wants | Invoke |
|---|---|
| Map the full codebase structure | iscape-map-codebase |
| Quick lightweight codebase scan | iscape-map-codebase --fast |
| Query mapped intelligence files | iscape-map-codebase --query |
| Generate a knowledge graph | iscape-graphify |
| Update project documentation | iscape-docs-update |
| Extract learnings from a completed phase | iscape-extract-learnings |

Invoke the matched skill directly using the Skill tool.
