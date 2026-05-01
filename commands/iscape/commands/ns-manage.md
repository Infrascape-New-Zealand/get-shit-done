---
name: iscape-manage
description: "config workspace | workstreams thread update ship inbox"
argument-hint: ""
allowed-tools:
  - Read
  - Skill
---

Route to the appropriate management skill based on the user's intent.
`iscape-config` (settings + advanced + integrations + profile) and `iscape-workspace`
(new + list + remove) are post-#2790 consolidated entries.

| User wants | Invoke |
|---|---|
| Configure Iscape settings (basic / advanced / integrations / profile) | iscape-config |
| Manage workspaces (create / list / remove) | iscape-workspace |
| Manage parallel workstreams | iscape-workstreams |
| Continue work in a fresh context thread | iscape-thread |
| Pause current work | iscape-pause-work |
| Resume paused work | iscape-resume-work |
| Update the Iscape installation | iscape-update |
| Ship completed work | iscape-ship |
| Process inbox items | iscape-inbox |
| Create a clean PR branch | iscape-pr-branch |
| Undo the last Iscape action | iscape-undo |

Invoke the matched skill directly using the Skill tool.
