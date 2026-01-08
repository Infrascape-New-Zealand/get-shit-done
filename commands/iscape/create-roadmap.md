---
name: iscape:create-roadmap
description: Create roadmap with phases for the project
allowed-tools:
  - Read
  - Write
  - Bash
  - AskUserQuestion
  - Glob
---

<objective>
Create project roadmap with phase breakdown.

Roadmaps define what work happens in what order. Run after /iscape:new-project.
</objective>

<execution_context>
@~/.claude/get-shit-done/workflows/create-roadmap.md
@~/.claude/get-shit-done/templates/roadmap.md
@~/.claude/get-shit-done/templates/state.md
</execution_context>

<context>
@context/PROJECT.md
@context/config.json
</context>

<process>

<step name="validate">
```bash
# Verify project exists
[ -f context/PROJECT.md ] || { echo "ERROR: No PROJECT.md found. Run /iscape:new-project first."; exit 1; }
```
</step>

<step name="check_existing">
Check if roadmap already exists:

```bash
[ -f context/ROADMAP.md ] && echo "ROADMAP_EXISTS" || echo "NO_ROADMAP"
```

**If ROADMAP_EXISTS:**
Use AskUserQuestion:
- header: "Roadmap exists"
- question: "A roadmap already exists. What would you like to do?"
- options:
  - "View existing" - Show current roadmap
  - "Replace" - Create new roadmap (will overwrite)
  - "Cancel" - Keep existing roadmap

If "View existing": `cat context/ROADMAP.md` and exit
If "Cancel": Exit
If "Replace": Continue with workflow
</step>

<step name="create_roadmap">
Follow the create-roadmap.md workflow starting from detect_domain step.

The workflow handles:
- Domain expertise detection
- Phase identification
- Research flags for each phase
- Confirmation gates (respecting config mode)
- ROADMAP.md creation
- STATE.md initialization
- Phase directory creation
- Git commit
</step>

<step name="done">
```
Roadmap created:
- Roadmap: context/ROADMAP.md
- State: context/STATE.md
- [N] phases defined

---

## ▶ Next Up

**Phase 1: [Name]** — [Goal from ROADMAP.md]

`/iscape:plan-phase 1`

<sub>`/clear` first → fresh context window</sub>

---

**Also available:**
- `/iscape:discuss-phase 1` — gather context first
- `/iscape:research-phase 1` — investigate unknowns
- Review roadmap

---
```
</step>

</process>

<output>
- `context/ROADMAP.md`
- `context/STATE.md`
- `context/phases/XX-name/` directories
</output>

<success_criteria>
- [ ] PROJECT.md validated
- [ ] ROADMAP.md created with phases
- [ ] STATE.md initialized
- [ ] Phase directories created
- [ ] Changes committed
</success_criteria>
