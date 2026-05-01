---
name: iscape-ideate
description: "exploration capture | explore sketch spike spec capture"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

Route to the appropriate exploration / capture skill based on the user's intent.
`iscape-note`, `iscape-add-todo`, `iscape-add-backlog`, and `iscape-plant-seed` were folded
into `iscape-capture` (with `--note`, default, `--backlog`, `--seed` modes) by
#2790. The capture target lists pending todos via `--list`.

| User wants | Invoke |
|---|---|
| Explore an idea or opportunity | iscape-explore |
| Sketch out a rough design or plan | iscape-sketch |
| Time-boxed technical spike | iscape-spike |
| Write a spec for a phase | iscape-spec-phase |
| Capture a thought (todo / note / backlog / seed) | iscape-capture |

Invoke the matched skill directly using the Skill tool.
