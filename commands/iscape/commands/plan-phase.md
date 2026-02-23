---
name: iscape:plan-phase
description: Create detailed execution plan for a phase (PLAN.md)
argument-hint: "[phase]"
allowed-tools:
  - Read
  - Bash
  - Write
  - Glob
  - Grep
  - AskUserQuestion
  - WebFetch
  - mcp__context7__*
---

<objective>
Create executable phase prompt with discovery, context injection, and task breakdown.

Purpose: Break down roadmap phases into concrete, executable PLAN.md files that Claude can execute.
Output: One or more PLAN.md files in the phase directory (context/phases/XX-name/{phase}-{plan}-PLAN.md)
</objective>

<execution_context>
@./workflows/plan-phase.md
@./templates/phase-prompt.md
@./references/plan-format.md
@./references/scope-estimation.md
@./references/checkpoints.md
@./references/tdd.md
</execution_context>

<context>
Phase number: $ARGUMENTS (optional - auto-detects next unplanned phase if not provided)

**Load project state first:**
@context/STATE.md

**Load roadmap:**
@context/ROADMAP.md

**Load phase context if exists (created by /iscape:discuss-phase):**
Check for and read `context/phases/XX-name/{phase}-CONTEXT.md` - contains research findings, clarifications, and decisions from phase discussion.

**Load codebase context if exists:**
Check for `context/codebase/` and load relevant documents based on phase type.
</context>

<process>
1. Check context/ directory exists (error if not - user should run /iscape:new-project)
2. If phase number provided via $ARGUMENTS, validate it exists in roadmap
3. If no phase number, detect next unplanned phase from roadmap
4. Follow plan-phase.md workflow:
   - Load project state and accumulated decisions
   - Perform mandatory discovery (Level 0-3 as appropriate)
   - Read project history (prior decisions, issues, concerns)
   - Break phase into tasks
   - Estimate scope and split into multiple plans if needed
   - Create PLAN.md file(s) with executable structure
</process>

<success_criteria>

- One or more PLAN.md files created in context/phases/XX-name/
- Each plan has: objective, execution_context, context, tasks, verification, success_criteria, output
- Tasks are specific enough for Claude to execute
- User knows next steps (execute plan or review/adjust)
  </success_criteria>
