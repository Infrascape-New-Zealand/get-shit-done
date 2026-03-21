# Planner Subagent Prompt Template

Template for spawning iscape-planner agent. The agent contains all planning expertise - this template provides planning context only.

---

## Template

```markdown
<planning_context>

**Phase:** {phase_number}
**Mode:** {standard | gap_closure}

**Project State:**
@context/STATE.md

**Roadmap:**
@context/ROADMAP.md

**Requirements (if exists):**
@context/REQUIREMENTS.md

**Phase Context (if exists):**
@context/phases/{phase_dir}/{phase_num}-CONTEXT.md

**Research (if exists):**
@context/phases/{phase_dir}/{phase_num}-RESEARCH.md

**Gap Closure (if --gaps mode):**
@context/phases/{phase_dir}/{phase_num}-VERIFICATION.md
@context/phases/{phase_dir}/{phase_num}-UAT.md

</planning_context>

<downstream_consumer>
Output consumed by /iscape:execute-phase
Plans must be executable prompts with:
- Frontmatter (wave, depends_on, files_modified, autonomous)
- Tasks in XML format
- Verification criteria
- must_haves for goal-backward verification
</downstream_consumer>

<quality_gate>
Before returning PLANNING COMPLETE:
- [ ] PLAN.md files created in phase directory
- [ ] Each plan has valid frontmatter
- [ ] Tasks are specific and actionable
- [ ] Dependencies correctly identified
- [ ] Waves assigned for parallel execution
- [ ] must_haves derived from phase goal
</quality_gate>
```

---

## Placeholders

| Placeholder | Source | Example |
|-------------|--------|---------|
| `{phase_number}` | From roadmap/arguments | `5` or `2.1` |
| `{phase_dir}` | Phase directory name | `05-user-profiles` |
| `{phase}` | Phase prefix | `05` |
| `{standard \| gap_closure}` | Mode flag | `standard` |

---

## Usage

**From /iscape:plan-phase (standard mode):**
```python
Task(
  prompt=filled_template,
  subagent_type="general-purpose",
  description="Plan Phase {phase}"
)
```

**From /iscape:plan-phase --gaps (gap closure mode):**
```python
Task(
  prompt=filled_template,  # with mode: gap_closure
  subagent_type="general-purpose",
  description="Plan gaps for Phase {phase}"
)
```

---

## Continuation

For checkpoints, spawn fresh agent with:

```markdown
<objective>
Continue planning for Phase {phase_number}: {phase_name}
</objective>

<prior_state>
Phase directory: @context/phases/{phase_dir}/
Existing plans: @context/phases/{phase_dir}/*-PLAN.md
</prior_state>

<checkpoint_response>
**Type:** {checkpoint_type}
**Response:** {user_response}
</checkpoint_response>

<mode>
Continue: {standard | gap_closure}
</mode>
```

---

**Note:** Planning methodology, task breakdown, dependency analysis, wave assignment, TDD detection, and goal-backward derivation are baked into the iscape-planner agent. This template only passes context.
