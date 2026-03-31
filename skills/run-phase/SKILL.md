---
name: run-phase
description: "Use when ready to execute a phase plan. Requires a path to the phase plan file. Orchestrates the full execution pipeline: implement chunks, review phase, generate docs, push stacked PRs."
---

# Run Phase

The entry point for executing an approved phase plan. Takes a phase plan file
and drives it through the full execution pipeline.

## Required Input

**This skill requires a path to a phase plan file.**

If no file path was provided, STOP immediately and ask:

> "Which phase plan should I execute? Provide the path to the phase plan file
> (e.g., `docs/plans/2026-03-31-feature-name.md`)."

Do not proceed without a valid file path. Do not guess. Do not search for
files. Ask the user.

If the provided file does not exist or is not a valid phase plan document,
STOP and report the error.

## What This Skill Does

This is the orchestrator for the entire execution side of the feature-workflow
system. It chains through the execution skills in order for a single phase.
If more phases remain after completion, it prompts you to run again with the
next phase.

**Announce at start:** "I'm using the run-phase skill to execute
`<file-path>`."

## The Execution Pipeline

```
run-phase <file-path> (you are here)
     │
     ├── 1. Validate input (file exists, is a phase plan)
     ├── 2. Invoke execute-phase (chunk-by-chunk with sub-agents + review)
     ├── 3. Invoke review-phase (full test suite, integration, acceptance)
     ├── 4. Invoke document-phase (changelog, API docs, README)
     └── 5. Invoke push-stacked-prs (one PR per chunk, stacked)
            │
            └── More phases? → "Run next phase with: run-phase <next-file>"
```

## Step 1: Validate Input

1. Confirm the file path was provided as an argument.
   - If missing: STOP and ask for it (see Required Input above).
2. Read the file and verify it is a phase plan document.
   - It should contain: phase objective, ordered chunk list, acceptance
     criteria, and chunk definitions with scope and affected files.
   - If the file is not a valid phase plan: STOP and report what is wrong.
3. Summarize the phase plan for the user:
   - Phase objective
   - Number of chunks
   - Key deliverables
   - Ask: "Ready to begin execution?"

Do not proceed until the user confirms.

## Step 2: Invoke Execute-Phase

Invoke the **execute-phase** skill. This will:
- Create a phase feature branch (via git-worktrees)
- For each chunk in order:
  - Dispatch a fresh sub-agent to implement the chunk
  - Run spec compliance review
  - Run code quality review
  - Present results for your approval (per chunk)
  - Commit on approval
- Dispatch a final code reviewer for the whole phase
- Finalize via finish-branch

**Cross-cutting skills active during execution:**
- **tdd** -- sub-agents follow Red-Green-Refactor
- **systematic-debugging** -- root cause before fixes
- **verify-completion** -- evidence before completion claims
- **code-review** -- two-stage review per chunk + final review

## Step 3: Invoke Review-Phase

After all chunks are committed, invoke the **review-phase** skill. This will:
- Run the full test suite
- Verify cross-chunk integration
- Review the cumulative phase diff
- Validate phase-level acceptance criteria
- Present a review report for your approval

**User decides:**
- **Approved** -- proceed to documentation
- **Issues found** -- loop back to execute-phase to address specific chunks
- **Abort** -- stop execution

## Step 4: Invoke Document-Phase

After phase approval, invoke the **document-phase** skill. This will:
- Generate changelog / release notes
- Update API documentation if applicable
- Update README / guides if applicable
- Present documentation for your approval before committing

## Step 5: Invoke Push-Stacked-PRs

After documentation is committed, invoke the **push-stacked-prs** skill.
This will:
- Create chunk branches from the phase branch
- Push to origin
- Create stacked PRs (one per chunk, each targeting the previous)
- Report the PR URLs

## After This Phase

If more phases remain in the feature plan, the skill will report:

> "Phase N complete. Next phase plan: `<path>`.
> To continue, invoke: run-phase `<next-phase-path>`"

If this was the final phase:

> "All phases complete. Feature fully implemented and pushed."

## Error Handling

| Situation | Action |
|---|---|
| No file path provided | STOP, ask for it |
| File does not exist | STOP, report error |
| File is not a valid phase plan | STOP, describe what is wrong |
| Tests fail during execution | execute-phase handles it (sub-agent fixes or escalates) |
| User rejects at any gate | Respect the decision, loop or stop as directed |
| Sub-agent is blocked | execute-phase handles it (re-dispatch or escalate) |

## Quick Reference

| Stage | Skill Invoked | Gate |
|---|---|---|
| Chunk implementation | execute-phase | Human review per chunk |
| Phase integration review | review-phase | Human approval |
| Documentation | document-phase | Human approval |
| PR creation | push-stacked-prs | -- |
