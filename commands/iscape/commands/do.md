---
name: iscape:do
description: Route freeform text to the right Iscape command automatically
argument-hint: "<description of what you want to do>"
allowed-tools:
  - Read
  - Bash
  - AskUserQuestion
---
<objective>
Analyze freeform natural language input and dispatch to the most appropriate Iscape command.

Acts as a smart dispatcher — never does the work itself. Matches intent to the best Iscape command using routing rules, confirms the match, then hands off.

Use when you know what you want but don't know which `/iscape:*` command to run.
</objective>

<execution_context>
@./workflows/do.md
@./references/ui-brand.md
</execution_context>

<context>
$ARGUMENTS
</context>

<process>
Execute the do workflow from @./workflows/do.md end-to-end.
Route user intent to the best Iscape command and invoke it.
</process>
