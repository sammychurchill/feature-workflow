---
name: plan-phases
description: >
  Use when you have an approved design spec and need to decompose it into a
  phased execution plan with PR-sized chunks, before touching code.
---

# Plan Phases

This skill implements stages 3-6 of the feature-workflow pipeline. It takes
an approved design spec and produces a concrete, phased execution plan.

> **Prerequisite**: An approved design spec must exist (produced by the
> brainstorm skill). If none exists, invoke brainstorm first.

---

## Stage 3: Complexity Gate

Assess the approved design along three dimensions:

| Dimension | Question |
|---|---|
| **Scope** | How many distinct functional areas does this touch? |
| **Risk** | How likely is a subtle breakage or integration issue? |
| **Components** | How many files, modules, or services are modified? |

### Scoring

| Score | Phases | Criteria |
|---|---|---|
| **Low** | 1 phase | Single concern, few files, low risk |
| **Medium** | 2-3 phases | Multiple concerns or moderate risk |
| **High** | 4+ phases | Cross-cutting, high risk, or many components |

State the score and reasoning before proceeding.

---

## Stage 4: Phase Planning

For each phase:

- [ ] **4.1 Decompose into N phases with clear boundaries**
  - Each phase has a single, well-defined goal.
  - Phases are ordered so earlier phases do not depend on later ones.

- [ ] **4.2 Map inter-phase dependencies**
  - Explicitly list what each phase requires from prior phases.
  - Identify any phases that could run in parallel.

- [ ] **4.3 Define acceptance criteria per phase**
  - Each phase has concrete, testable acceptance criteria.
  - Criteria must be verifiable without subjective judgment.

- [ ] **4.4 Break each phase into PR-sized chunks**
  - Each chunk = 1 future PR.
  - Each chunk is **independently mergeable** and leaves the codebase
    in a working state (all tests pass, no broken imports, no dead code).
  - Chunks within a phase are ordered by dependency.

### Chunk Requirements

Every chunk must specify:
- **Goal**: One sentence describing what this chunk accomplishes.
- **Files**: Exact file paths that will be created or modified.
- **Steps**: Bite-sized steps (2-5 minutes each) with complete code —
  no placeholders, no "similar to chunk N", no TBD.
- **Tests**: What tests to write or update (TDD — tests first).
- **Acceptance criteria**: How to verify this chunk is done.

---

## Stage 5: Update Design Spec

- [ ] **5.1 Compile phase overviews**
  - For each phase: name, goal, chunk count, key risks.

- [ ] **5.2 Inject into design spec**
  - Append a "Phases" section to the existing design spec file
    (the one created during brainstorm).

---

## Stage 6: Plan Review Gate

- [ ] **6.1 Write plan document**
  - Save to: `docs/plans/YYYY-MM-DD-<feature-name>.md`
    (replace YYYY-MM-DD with today's date and `<feature-name>` with a
    short kebab-case label).

### Plan Document Structure

```markdown
# Plan: <Feature Name>

## Goal
<One-paragraph summary of what we are building and why.>

## Architecture
<High-level architecture description. Reference the design spec.>

## Tech Stack
<Languages, frameworks, libraries involved.>

## Agentic Execution Note
Each chunk below is designed to be executed by a fresh subagent via the
execute-phase skill. See that skill for execution protocol.

## Phase 1: <Phase Name>

### Chunk 1.1: <Chunk Title>
**Goal:** <one sentence>
**Files:**
- `path/to/file.ts` (modify)
- `path/to/new-file.ts` (create)

**Steps:**
- [ ] Step 1: <exact description with complete code>
- [ ] Step 2: <exact description with complete code>

**Tests:**
- [ ] <test description>

**Acceptance Criteria:**
- [ ] <criterion>

### Chunk 1.2: <Chunk Title>
...

## Phase 2: <Phase Name>
...
```

### Plan Self-Review

After writing the plan, verify:
- [ ] **Spec coverage**: Every requirement from the design spec is addressed
  by at least one chunk.
- [ ] **Placeholder scan**: Search for TBD, TODO, "implement later",
  "similar to chunk N", or any vague language. Remove or resolve every one.
- [ ] **Type consistency**: All referenced types, interfaces, and function
  signatures are consistent across chunks.
- [ ] **Dependency ordering**: No chunk references work from a later chunk.
- [ ] **Working codebase**: Each chunk, if merged alone, leaves the codebase
  in a working state.

### Principles Embedded in Every Plan
- **DRY** — Do not repeat yourself across chunks.
- **YAGNI** — Cut anything not needed for the immediate goal.
- **TDD** — Tests are written before implementation in every chunk.
- **Frequent commits** — Each step within a chunk should be committable.

- [ ] **6.2 Present plan to the user**
  - Summarize the plan in the conversation.
  - Point the user to the written file for full details.
  - Ask the user to review and choose one of:
    - **Approved** — Plan is accepted. Ready for execution.
    - **Revise** — User has feedback. Loop back to the relevant stage
      (3, 4, or 5) and iterate.
    - **Rejected** — User wants to stop. End the workflow.

- [ ] **6.3 On Approval: offer execution choice**
  - Once the user approves the plan, present two options:

  > **How would you like to execute this plan?**
  >
  > 1. **Subagent-Driven** (recommended) — A fresh subagent is dispatched
  >    for each chunk via the execute-phase skill. Each subagent gets a
  >    clean context with only the chunk details and relevant code. Best
  >    for larger plans or when context window freshness matters.
  >
  > 2. **Inline Execution** — Execute chunks sequentially in this current
  >    session. Simpler but uses more context window. Best for small plans
  >    (1-2 chunks).

  - Regardless of choice, execution proceeds chunk by chunk through the
    execute-phase skill.

---

## Terminal State

This skill always ends in one of:
1. **Approved** → offer execution choice, then begin execution via
   execute-phase.
2. **Revise** → loop within this skill until approved or rejected.
3. **Rejected** → stop. Inform the user the workflow has ended.
