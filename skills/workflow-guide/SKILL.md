---
name: workflow-guide
description: >
  Use at session start and before any action to check all available skills
  for applicability. Skills override default behavior; user instructions
  (CLAUDE.md) take highest priority.
---

# Workflow Guide

You are operating inside the **feature-workflow** system. Every user message
MUST be evaluated against the available skills before you do anything else.

## Core Rule

> If there is even a **1% chance** that a skill applies to the current message,
> you **MUST** invoke it via the Skill tool. Do not skip skills.

## Priority Order

1. **User instructions** (CLAUDE.md) — highest priority, always wins.
2. **Skills** — override default Claude Code behavior when invoked.
3. **Default behavior** — only used when no skill applies and no user
   instruction is relevant.

## Skill Invocation Flowchart

```
User message received
        │
        ▼
┌───────────────────┐
│ Check all skills  │
│ for applicability │
└───────┬───────────┘
        │
        ▼
   ┌─────────┐       ┌──────────────────────┐
   │ Match?  │──No──▶│ Proceed with default │
   └────┬────┘       │ behavior             │
        │Yes         └──────────────────────┘
        ▼
┌───────────────────┐
│ Announce which    │
│ skill is being    │
│ invoked and why   │
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│ Invoke skill via  │
│ the Skill tool    │
└───────┬───────────┘
        │
        ▼
┌───────────────────┐
│ Follow the skill  │
│ instructions fully│
└───────────────────┘
```

## Evaluation Order for Skills

Process skills in this order:

1. **Process skills first** — brainstorm, systematic-debugging, receive-feedback
2. **Implementation skills second** — execute-phase, tdd, review-phase,
   document-phase, push-stacked-prs, finish-branch
3. **Utility skills** — git-worktrees, parallel-agents, code-review,
   verify-completion

If multiple skills apply, invoke the highest-priority one first. The skill
itself will chain to the next skill when appropriate.

## Subagent Exception

Subagents dispatched for a specific, scoped task (e.g., via execute-phase or
parallel-agents) should **skip** this workflow-guide skill. They already have
their task defined and should execute it directly.

## Red Flags — Rationalizations That Must Be Overridden

If you catch yourself thinking any of the following, **stop and invoke the
applicable skill anyway**:

| Rationalization | Why It Is Wrong |
|---|---|
| "This is too simple for a skill" | Every project goes through brainstorm. No exceptions. |
| "Let me just explore first" | Exploration IS brainstorming. Invoke the brainstorm skill. |
| "I already know how to do this" | The skill enforces structure and gates that prevent mistakes. |
| "The user just wants a quick fix" | Quick fixes still need the systematic-debugging skill. |
| "I'll use the skill after I look around" | Looking around without a skill is unstructured. Invoke first. |
| "This doesn't need a plan" | plan-phases exists precisely because you think it doesn't. |
| "I'll just make the change directly" | execute-phase ensures reviewable, mergeable increments. |
| "The user didn't ask for a skill" | Users don't invoke skills — you do, automatically. |

## Entry Points

Most users interact with the system through these two skills:

- **plan-feature** — Start here for any new feature or significant change.
  Chains through brainstorm → plan-phases automatically.
- **run-phase** `<file-path>` — Execute an approved phase plan. Requires
  the path to a phase plan file. Chains through execute-phase → review-phase
  → document-phase → push-stacked-prs automatically.

## Available Skills

### Entry Points
- **plan-feature** — Orchestrates the full planning pipeline
- **run-phase** — Orchestrates the full execution pipeline for one phase

### Planning (chained by plan-feature)
- **brainstorm** — Design and spec refinement before implementation
- **plan-phases** — Decompose approved design into phased execution plan

### Execution (chained by run-phase)
- **execute-phase** — Execute a single phase from the plan
- **review-phase** — Review completed phase output
- **document-phase** — Generate documentation for a phase
- **push-stacked-prs** — Push stacked PRs for completed work

### Cross-Cutting (activate automatically when conditions are met)
- **tdd** — Test-driven development workflow
- **systematic-debugging** — Structured debugging process
- **verify-completion** — Verify all acceptance criteria are met
- **code-review** — Review code for quality and correctness
- **receive-feedback** — Process and incorporate user feedback
- **git-worktrees** — Manage git worktrees for parallel work
- **parallel-agents** — Dispatch parallel subagents for independent tasks
- **finish-branch** — Finalize and clean up a feature branch
