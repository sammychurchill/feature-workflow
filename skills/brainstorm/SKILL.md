---
name: brainstorm
description: >
  Use before any creative work -- new feature requests, design discussions,
  or any implementation request that lacks an approved design spec. Explores
  user intent, requirements, and design before implementation.
---

# Brainstorm

This skill implements the first two stages of the feature-workflow pipeline:
**Brainstorming** and **Design Review Gate**.

> **HARD-GATE**: You CANNOT invoke any implementation skill (execute-phase,
> tdd, push-stacked-prs, etc.) until the design spec produced by this skill
> has been explicitly approved by the user. No exceptions.

## Interaction Style

- Ask **one question at a time**.
- Prefer **multiple choice** when possible (offer 2-4 options).
- Apply **YAGNI ruthlessly** — cut anything that is not needed for the
  immediate goal.
- Every project goes through this, even ones that seem "simple."

---

## Stage 1: Brainstorming

Complete these steps in order. Do not skip any.

### Checklist

- [ ] **1.1 Gather and clarify requirements**
  - Ask the user what they want to build and why.
  - Identify the target users, key use cases, and success criteria.
  - One question at a time. Multiple choice preferred.
  - Continue until requirements are clear and unambiguous.

- [ ] **1.2 Research existing codebase**
  - Search for related code, patterns, conventions, and constraints.
  - Identify relevant files, modules, and dependencies.
  - Note any existing patterns that the new work must follow.
  - Surface any constraints (performance, compatibility, API contracts).

- [ ] **1.3 Ideate solutions**
  - Propose **2-3 approaches** with explicit tradeoffs.
  - For each approach, state: summary, pros, cons, estimated complexity.
  - Ask the user which direction they prefer (or if they want a hybrid).

- [ ] **1.4 Draft design spec**
  - Write the spec to: `docs/specs/YYYY-MM-DD-<topic>-design.md`
    (replace YYYY-MM-DD with today's date and `<topic>` with a short
    kebab-case label).
  - The spec must include:
    - **Problem Statement** — What problem are we solving and why.
    - **Proposed Solution** — The chosen approach in detail.
    - **Scope** — What is in scope and what is explicitly out of scope.
    - **Tradeoffs** — What we are giving up and why that is acceptable.
    - **Open Questions** — Anything still unresolved (should be empty
      before approval).

- [ ] **1.5 Self-validate the spec**
  - Run through this checklist against the written spec:
    - **Placeholder scan**: Search for TBD, TODO, "implement later",
      "similar to X", or any vague language. Remove or resolve every one.
    - **Consistency check**: Do all sections agree with each other? Are
      there contradictions between scope and solution?
    - **Scope check**: Is anything included that violates YAGNI? Cut it.
    - **Ambiguity check**: Could a different engineer read this spec and
      reach a different conclusion about what to build? If yes, clarify.
  - Fix any issues found before proceeding.

---

## Stage 2: Design Review Gate

- [ ] **2.1 Present the validated spec to the user**
  - Summarize the spec in the conversation.
  - Point the user to the written file for full details.
  - Ask the user to review and choose one of:
    - **Approved** — Design is accepted. Proceed to planning.
    - **Revise** — User has feedback. Loop back to the relevant step
      in Stage 1 (1.1-1.5) and iterate.
    - **Rejected** — User wants to stop. End the workflow.

- [ ] **2.2 On Approval: invoke plan-phases**
  - Once the user says "approved" (or equivalent affirmative), the
    **only** next skill allowed is **plan-phases**.
  - Invoke plan-phases via the Skill tool immediately.
  - Do NOT invoke any other skill. Do NOT start implementing.

---

## Terminal State

This skill always ends in one of:
1. **Approved** → invoke `plan-phases` skill (the only permitted next step).
2. **Revise** → loop within this skill until approved or rejected.
3. **Rejected** → stop. Inform the user the workflow has ended.
