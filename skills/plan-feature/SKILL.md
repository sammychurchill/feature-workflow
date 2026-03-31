---
name: plan-feature
description: "Use when starting any new feature, project, or significant change. The primary entry point for the feature-workflow system -- kicks off brainstorming, design, and phase planning."
---

# Plan Feature

The entry point for turning an idea into an approved, phased execution plan.

**Announce at start:** "I'm using the plan-feature skill to start the planning workflow."

## What This Skill Does

This is the orchestrator for the entire planning side of the feature-workflow
system. It sets up the session context and chains through the planning skills
in order. You do not need to invoke the individual skills yourself -- this
skill handles the sequencing.

## The Planning Pipeline

```
plan-feature (you are here)
     │
     ├── 1. Load workflow context (skill catalog, priorities, rules)
     ├── 2. Invoke brainstorm (requirements → design spec → user approval)
     └── 3. Invoke plan-phases (complexity → phases → chunks → user approval)
            │
            └── Output: Approved phase plan, ready for run-phase
```

## Step 1: Establish Workflow Context

Before doing anything else, internalize these rules for the session:

**Skill priority:**
1. User instructions (CLAUDE.md) -- highest, always wins
2. Feature-workflow skills -- override default behavior
3. Default Claude Code behavior -- lowest

**Cross-cutting skills active throughout this session:**
- **tdd** -- all implementation follows Red-Green-Refactor
- **systematic-debugging** -- root cause before fixes, always
- **verify-completion** -- evidence before claims, always
- **code-review** -- review before merge, always

**If you catch yourself about to skip a skill, stop.** The skill exists
because skipping it causes problems. Even "simple" projects go through the
full pipeline.

## Step 2: Invoke Brainstorm

Invoke the **brainstorm** skill. This will:
- Gather and clarify requirements (one question at a time)
- Research the existing codebase
- Propose 2-3 approaches with tradeoffs
- Draft a design spec
- Self-validate the spec
- Present it for your approval

**You must approve the design before anything else happens.**

The brainstorm skill handles the Design Review Gate:
- **Approved** -- proceeds to Step 3 automatically
- **Revise** -- loops within brainstorm until you approve or reject
- **Rejected** -- workflow stops

## Step 3: Invoke Plan-Phases

After design approval, the **plan-phases** skill is invoked automatically.
This will:
- Score complexity (Low / Medium / High)
- Decompose into phases with clear boundaries
- Break each phase into PR-sized chunks
- Write phase plan documents
- Present everything for your approval

**You must approve the phase plan before execution can begin.**

The plan-phases skill handles the Plan Review Gate:
- **Approved** -- outputs are ready for run-phase
- **Revise** -- loops within plan-phases
- **Rejected** -- workflow stops

## When Planning Is Complete

After both approvals, plan-phases will tell you the plan is ready and where
it was saved. To execute it, use the **run-phase** skill with the path to the
phase plan file.

Example:
```
Planning complete. Phase plan saved to docs/plans/2026-03-31-auth-system.md

To execute phase 1, invoke: run-phase docs/plans/2026-03-31-auth-system.md
```

## Quick Reference

| Stage | Skill | Gate |
|---|---|---|
| Requirements + Design | brainstorm | Design Review (user approves) |
| Phases + Chunks | plan-phases | Plan Review (user approves) |
| Execution | run-phase | (separate skill) |
