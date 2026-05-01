# Iscape Canonical Artifact Registry

This directory contains the template files for every artifact that Iscape workflows officially produce. The table below is the authoritative index: **if a `context/` root file is not listed here, `iscape-health` will flag it as W019** (unrecognized artifact).

Agents should query this file before treating a `context/` file as authoritative. If the file name does not appear below, it is not a canonical Iscape artifact.

---

## `context/` Root Artifacts

These files live directly at `context/` — not inside phase subdirectories.

| File | Template | Produced by | Purpose |
|------|----------|-------------|---------|
| `PROJECT.md` | `project.md` | `/iscape-new-project` | Project identity, goals, requirements summary |
| `ROADMAP.md` | `roadmap.md` | `/iscape-new-milestone`, `/iscape-new-project` | Phase plan with milestones and progress tracking |
| `STATE.md` | `state.md` | `/iscape-new-project`, `/iscape-health --repair` | Current session state, active phase, last activity |
| `REQUIREMENTS.md` | `requirements.md` | `/iscape-new-milestone` | Functional requirements with traceability |
| `MILESTONES.md` | `milestone.md` | `/iscape-complete-milestone` | Log of completed milestones with accomplishments |
| `BACKLOG.md` | *(inline)* | `/iscape-add-backlog` | Pending ideas and deferred work |
| `LEARNINGS.md` | *(inline)* | `/iscape-extract-learnings`, `/iscape-execute-phase` | Phase retrospective learnings for future plans |
| `THREADS.md` | *(inline)* | `/iscape-thread` | Persistent discussion threads |
| `config.json` | `config.json` | `/iscape-new-project`, `/iscape-health --repair` | Project-specific Iscape configuration |
| `CLAUDE.md` | `claude-md.md` | `/iscape-profile` | Auto-assembled Claude Code context file |

### Version-stamped artifacts (pattern: `vX.Y-*.md`)

| Pattern | Produced by | Purpose |
|---------|-------------|---------|
| `vX.Y-MILESTONE-AUDIT.md` | `/iscape-audit-milestone` | Milestone audit report before archiving |

These files are archived to `context/milestones/` by `/iscape-complete-milestone`. Finding them at the `context/` root after completion indicates the archive step was skipped.

---

## Phase Subdirectory Artifacts (`context/phases/NN-name/`)

These files live inside a phase directory. They are NOT checked by W019 (which only inspects the `context/` root).

| File Pattern | Template | Produced by | Purpose |
|-------------|----------|-------------|---------|
| `NN-MM-PLAN.md` | `phase-prompt.md` | `/iscape-plan-phase` | Executable implementation plan |
| `NN-MM-SUMMARY.md` | `summary.md` | `/iscape-execute-phase` | Post-execution summary with learnings |
| `NN-CONTEXT.md` | `context.md` | `/iscape-discuss-phase` | Scoped discussion decisions for the phase |
| `NN-RESEARCH.md` | `research.md` | `/iscape-research-phase`, `/iscape-plan-phase` | Technical research for the phase |
| `NN-VALIDATION.md` | `VALIDATION.md` | `/iscape-research-phase` (Nyquist) | Validation architecture (Nyquist method) |
| `NN-UAT.md` | `UAT.md` | `/iscape-validate-phase` | User acceptance test results |
| `NN-PATTERNS.md` | *(inline)* | `/iscape-plan-phase` (pattern mapper) | Analog file mapping for the phase |
| `NN-UI-SPEC.md` | `UI-SPEC.md` | `/iscape-ui-phase` | UI design contract |
| `NN-SECURITY.md` | `SECURITY.md` | `/iscape-secure-phase` | Security threat model |
| `NN-AI-SPEC.md` | `AI-SPEC.md` | `/iscape-ai-integration-phase` | AI integration spec with eval strategy |
| `NN-DEBUG.md` | `DEBUG.md` | `/iscape-debug` | Debug session log |
| `NN-REVIEWS.md` | *(inline)* | `/iscape-review` | Cross-AI review feedback |

---

## Milestone Archive (`context/milestones/`)

Files archived by `/iscape-complete-milestone`. These are never checked by W019.

| File Pattern | Source |
|-------------|--------|
| `vX.Y-ROADMAP.md` | Snapshot of ROADMAP.md at milestone close |
| `vX.Y-REQUIREMENTS.md` | Snapshot of REQUIREMENTS.md at milestone close |
| `vX.Y-MILESTONE-AUDIT.md` | Moved from `context/` root |
| `vX.Y-phases/` | Archived phase directories (if `--archive-phases` used) |

---

## Adding a New Canonical Artifact

When a new workflow produces a `context/` root file:

1. Add the file name to `CANONICAL_EXACT` in `get-shit-done/bin/lib/artifacts.cjs`
2. Add a row to the **`context/` Root Artifacts** table above
3. Add the template to `get-shit-done/templates/` if one exists
