# Model Profiles

Model profiles control which Claude model each Iscape agent uses. This allows balancing quality vs token spend.

## Profile Definitions

| Agent | `quality` | `balanced` | `budget` |
|-------|-----------|------------|----------|
| **Planning & Roadmapping** | | | |
| iscape-planner | opus | opus | sonnet |
| iscape-roadmapper | opus | sonnet | sonnet |
| **Execution** | | | |
| iscape-executor | opus | sonnet | sonnet |
| iscape-developer | opus | sonnet | sonnet |
| **Research** | | | |
| iscape-phase-researcher | opus | sonnet | haiku |
| iscape-project-researcher | opus | sonnet | haiku |
| iscape-research-synthesizer | sonnet | sonnet | haiku |
| iscape-ai-researcher | opus | sonnet | haiku |
| iscape-domain-researcher | opus | sonnet | haiku |
| iscape-advisor-researcher | sonnet | sonnet | haiku |
| iscape-assumptions-analyzer | sonnet | sonnet | haiku |
| **Design & Specification** | | | |
| iscape-ui-researcher | opus | sonnet | sonnet |
| iscape-eval-planner | opus | sonnet | sonnet |
| iscape-framework-selector | sonnet | sonnet | sonnet |
| **Verification & Auditing** | | | |
| iscape-verifier | sonnet | sonnet | haiku |
| iscape-integration-checker | sonnet | sonnet | haiku |
| iscape-plan-checker | sonnet | sonnet | haiku |
| iscape-security-auditor | opus | sonnet | sonnet |
| iscape-ui-checker | sonnet | sonnet | haiku |
| iscape-ui-auditor | sonnet | sonnet | haiku |
| iscape-eval-auditor | sonnet | sonnet | haiku |
| iscape-nyquist-auditor | sonnet | sonnet | haiku |
| iscape-code-reviewer | opus | sonnet | sonnet |
| **Code & Doc Execution** | | | |
| iscape-code-fixer | sonnet | sonnet | sonnet |
| iscape-doc-writer | sonnet | sonnet | haiku |
| iscape-doc-synthesizer | sonnet | sonnet | haiku |
| iscape-doc-verifier | sonnet | haiku | haiku |
| iscape-doc-classifier | sonnet | haiku | haiku |
| **Debugging** | | | |
| iscape-debugger | opus | sonnet | sonnet |
| iscape-debug-session-manager | sonnet | sonnet | sonnet |
| **Codebase Analysis** | | | |
| iscape-codebase-mapper | sonnet | haiku | haiku |
| iscape-intel-updater | sonnet | haiku | haiku |
| iscape-pattern-mapper | sonnet | haiku | haiku |
| **Profiling** | | | |
| iscape-user-profiler | sonnet | sonnet | haiku |

## Profile Philosophy

**quality** - Maximum reasoning power
- Opus for all decision-making agents
- Sonnet for read-only verification
- Use when: quota available, critical architecture work

**balanced** (default) - Smart allocation
- Opus only for planning (where architecture decisions happen)
- Sonnet for execution and research (follows explicit instructions)
- Sonnet for verification (needs reasoning, not just pattern matching)
- Use when: normal development, good balance of quality and cost

**budget** - Minimal Opus usage
- Sonnet for anything that writes code
- Haiku for research and verification
- Use when: conserving quota, high-volume work, less critical phases

## Resolution Logic

Orchestrators resolve model before spawning:

```
1. Read context/config.json
2. Check model_overrides for agent-specific override
3. If no override, look up agent in profile table
4. Pass model parameter to Task call
```

## Team Member Model Overrides

When using a dev team (`team.enabled: true`), individual members can override the profile default via `team.member_model`. This applies to all developers in the team equally.

For per-member overrides, use `model_overrides`:

```json
{
  "model_profile": "balanced",
  "team": { "enabled": true, "size": 3, "member_model": "opus" }
}
```

`team.member_model` resolution priority:
1. `team.member_model` (if non-null) — overrides profile for all developers
2. `model_overrides["iscape-developer"]` (if set) — agent-specific override
3. Profile table lookup for `iscape-developer`

## Per-Agent Overrides

Override specific agents without changing the entire profile:

```json
{
  "model_profile": "balanced",
  "model_overrides": {
    "iscape-executor": "opus",
    "iscape-planner": "haiku"
  }
}
```

Overrides take precedence over the profile. Valid values: `opus`, `sonnet`, `haiku`.

## Switching Profiles

Runtime: `/iscape:set-profile <profile>`

Per-project default: Set in `context/config.json`:
```json
{
  "model_profile": "balanced"
}
```

## Design Rationale

**Why Opus for iscape-planner?**
Planning involves architecture decisions, goal decomposition, and task design. This is where model quality has the highest impact.

**Why Sonnet for iscape-executor?**
Executors follow explicit PLAN.md instructions. The plan already contains the reasoning; execution is implementation.

**Why Sonnet (not Haiku) for verifiers in balanced?**
Verification requires goal-backward reasoning - checking if code *delivers* what the phase promised, not just pattern matching. Sonnet handles this well; Haiku may miss subtle gaps.

**Why Haiku for iscape-codebase-mapper?**
Read-only exploration and pattern extraction. No reasoning required, just structured output from file contents.

**Why `inherit` instead of passing `opus` directly?**
Claude Code's `"opus"` alias maps to a specific model version. Organizations may block older opus versions while allowing newer ones. Iscape returns `"inherit"` for opus-tier agents, causing them to use whatever opus version the user has configured in their session. This avoids version conflicts and silent fallbacks to Sonnet.

**Why Opus for iscape-ui-researcher and iscape-eval-planner?**
Both produce design contracts (UI-SPEC.md, AI-SPEC.md) that constrain all downstream work. Getting these wrong is expensive. Same reasoning as iscape-planner.

**Why Opus for iscape-security-auditor and iscape-code-reviewer?**
Security auditing requires reasoning about threat models and implicit attack surfaces — pattern matching is insufficient. Code review at quality tier benefits from Opus for catching subtle correctness bugs and security implications, not just style.

**Why Sonnet for iscape-code-fixer across all profiles?**
Fixes follow a REVIEW.md checklist — the reasoning is already done. This is implementation work, same as iscape-executor.

**Why Haiku for iscape-doc-classifier, iscape-doc-verifier, iscape-intel-updater, and iscape-pattern-mapper?**
All four are structured extraction tasks: classify a doc, verify claims, write intel entries, map patterns. No architecture decisions involved. Haiku handles structured extraction well at lower cost.
