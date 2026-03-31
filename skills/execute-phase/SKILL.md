---
name: execute-phase
description: >
  Use when executing a phase plan with independent chunks in the current
  session. Dispatches fresh subagents per chunk with two-stage review and
  human approval gates.
---

# Execute Phase

Execute a phase plan by dispatching a fresh subagent per chunk, with two-stage
review after each and a human approval gate before committing.

**Announce at start:** "I'm using the execute-phase skill to implement this
phase plan."

**Core principle:** Fresh subagent per chunk + two-stage review (spec then
quality) + human approval gate = high quality, reviewable increments.

## When to Use

Use this skill when you have a Phase Plan Document (produced by plan-phases)
and are ready to implement it. The phase plan contains an ordered list of
chunks, each mapping to a single PR.

## The Process

### Stage 1: Phase Initialization

1. **Load** the Phase Plan Document. Parse the ordered chunk list with full
   definitions (scope, affected files, acceptance criteria, dependencies).
2. **Create phase feature branch** using the git-worktrees skill. Never start
   on main/master without explicit user consent.
3. **Build task queue** from the ordered chunk list. Record chunk order,
   dependencies, and acceptance criteria.

### Stage 2: Task Loop (per Chunk)

Process chunks sequentially. Each chunk is executed by a fresh subagent to
prevent context pollution.

**Single implementer per chunk.** Do not dispatch multiple implementation
subagents in parallel -- they will conflict with each other.

For each chunk, dispatch a subagent using the `./chunk-implementer.md` prompt
template with the following stages:

#### a. Preflight

The subagent runs the existing test suite before making any changes.

- If tests **pass**: codebase is healthy, proceed.
- If tests **fail** before any changes: abort immediately and report failure to
  main thread. Something is broken upstream and must be fixed first.

#### b. Load Context

Provide the subagent with all relevant context upfront. **Do not make the
subagent read files** -- paste the full text into the prompt:

- **Design spec** -- the validated design spec for big-picture understanding
- **Phase plan** -- the current phase scope and structure
- **Current chunk definition** -- the specific scope, affected files,
  acceptance criteria for this chunk
- **Prior chunk outcomes** -- what was done in previous chunks (summaries of
  changes, any concerns raised)

#### c. Implement

The subagent:

1. Writes code to fulfill the chunk scope
2. Writes or updates tests
3. Runs tests and iterates until passing
4. Follows TDD practices (via the tdd skill patterns)

#### d. QA and Review

Multi-pass review before returning to main thread:

1. **Self-QA** -- validate implementation against the chunk's acceptance
   criteria
2. **Lint and static analysis** -- run linters and configured analysis tools
3. **Security review** -- check for common vulnerabilities (injection, auth
   issues, secrets in code, etc.)
4. **Diff self-review** -- review own diff as a final sanity check

If any issues are found, loop back to implement and fix before returning.

#### e. Return Results

The subagent packages and returns:

- Status: `DONE` | `DONE_WITH_CONCERNS` | `NEEDS_CONTEXT` | `BLOCKED`
- Summary of what was implemented
- Test results
- Files changed
- Self-review findings
- Any concerns or issues

After the implementer returns, dispatch two review subagents in sequence:

1. **Spec compliance review** (using `./spec-reviewer.md`): Verify the
   implementation matches the chunk specification -- nothing missing, nothing
   extra. If issues found, send back to implementer to fix, then re-review.

2. **Code quality review** (using `./quality-reviewer.md`): Only after spec
   compliance passes. Review for code quality, maintainability, test coverage.
   If issues found, send back to implementer to fix, then re-review.

**Never start code quality review before spec compliance passes.**

### Stage 3: Human Review (per Chunk)

Present chunk results to the user:

- The diff (complete changes for this chunk)
- Test results (what was tested, pass/fail counts)
- Review notes (from both spec and quality reviewers)
- Any concerns raised by the implementer

**User decides:**

| Decision | Action |
|---|---|
| **Approved** | Commit changes to the phase branch. Move to next chunk. |
| **Changes Requested** | Re-run the chunk with user feedback incorporated. Dispatch a fresh subagent with the feedback as additional context. |
| **Abort** | Stop execution of this phase. Do not commit. |

### After All Chunks

Once all chunks are committed:

1. Dispatch a final code reviewer subagent (using the code-review skill) to
   review the entire phase implementation as a whole -- not just individual
   chunks, but how they integrate together.
2. Use the finish-branch skill to finalize the phase branch.

## Model Selection

Use the least powerful model that can handle each role to conserve cost and
increase speed.

| Complexity Signal | Model Tier | Examples |
|---|---|---|
| Touches 1-2 files with complete spec | Cheap / fast | Isolated function, clear input/output, mechanical |
| Multi-file coordination, integration | Standard | Cross-module changes, pattern matching, debugging |
| Architecture, design, broad codebase | Most capable | Review tasks, design judgment, complex refactoring |

**Task complexity signals:**

- Chunk affects 1-2 files with a complete spec and clear acceptance criteria
  --> cheap model
- Chunk touches multiple files with integration concerns or ambiguous
  boundaries --> standard model
- Chunk requires design judgment or broad codebase understanding --> most
  capable model
- All review tasks (spec compliance, code quality, final review) --> most
  capable model

## Handling Implementer Status

**DONE:** Proceed to spec compliance review.

**DONE_WITH_CONCERNS:** The implementer completed the work but flagged doubts.
Read the concerns before proceeding. If concerns are about correctness or
scope, address them before review. If they are observations (e.g., "this file
is getting large"), note them and proceed to review.

**NEEDS_CONTEXT:** The implementer needs information that was not provided.
Provide the missing context and re-dispatch.

**BLOCKED:** The implementer cannot complete the task. Assess the blocker:

1. If it is a context problem, provide more context and re-dispatch with the
   same model.
2. If the task requires more reasoning, re-dispatch with a more capable model.
3. If the task is too large, break it into smaller pieces.
4. If the plan itself is wrong, escalate to the user.

**Never** ignore an escalation or force the same model to retry without
changes. If the implementer said it is stuck, something needs to change.

## Prompt Templates

- `./chunk-implementer.md` -- Dispatch chunk implementer subagent
- `./spec-reviewer.md` -- Dispatch spec compliance reviewer subagent
- `./quality-reviewer.md` -- Dispatch code quality reviewer subagent

**Provide full task text in every prompt.** Do not make subagents read plan
files. You are the controller -- you curate exactly what context each subagent
needs and paste it directly into the prompt.

## Example Workflow

```
You: I'm using the execute-phase skill to implement this phase plan.

[Load phase plan: docs/plans/auth-system-phase-1.md]
[Parse 4 chunks: session-store, token-validation, middleware, integration-tests]
[Create phase branch via git-worktrees: feature/auth-phase-1]

--- Chunk 1: Session Store ---

[Build prompt with: design spec + phase plan + chunk-1 definition + no prior outcomes]
[Dispatch implementer subagent (cheap model -- 2 files, clear spec)]

Implementer reports: DONE
  - Created session_store.py with Redis backend
  - 12 tests, all passing
  - Self-review: clean

[Dispatch spec reviewer]
Spec reviewer: Spec compliant -- all requirements met

[Dispatch quality reviewer]
Quality reviewer: Approved -- clean code, good test coverage

[Present to user]
User: Approved

[Commit chunk-1 to phase branch]

--- Chunk 2: Token Validation ---

[Build prompt with: design spec + phase plan + chunk-2 definition + chunk-1 outcome]
[Dispatch implementer subagent (standard model -- 4 files, integration concerns)]

Implementer reports: DONE_WITH_CONCERNS
  - Concern: "JWT library has a known CVE, should we pin version?"

[Address concern: check CVE, decide with user, provide guidance]
[Re-dispatch or proceed based on resolution]

[Dispatch spec reviewer]
Spec reviewer: Issues found -- missing refresh token rotation

[Implementer fixes refresh token rotation]
[Spec reviewer re-reviews: Spec compliant]

[Dispatch quality reviewer]
Quality reviewer: Approved

[Present to user]
User: Approved

[Commit chunk-2 to phase branch]

--- Chunk 3: Middleware ---

[Continue pattern...]

--- Chunk 4: Integration Tests ---

[Continue pattern...]

--- All chunks committed ---

[Dispatch final code reviewer for entire phase]
Final reviewer: Integration looks good, all components connect properly

[Use finish-branch skill to finalize]

Done -- phase branch ready for review-phase.
```

## Red Flags

**Never:**

- Start implementation on main/master branch without explicit user consent
- Skip reviews (spec compliance OR code quality) for any chunk
- Proceed with unfixed issues from either reviewer
- Dispatch multiple implementation subagents in parallel (they will conflict)
- Make subagents read plan files (provide full text in the prompt)
- Skip context loading (subagent needs design spec, phase plan, chunk
  definition, and prior outcomes)
- Ignore subagent questions or concerns (answer before letting them proceed)
- Accept "close enough" on spec compliance (issues found = not done)
- Skip review loops (reviewer found issues = implementer fixes = review again)
- Let implementer self-review replace actual review (both are needed)
- Start code quality review before spec compliance passes
- Move to the next chunk while either review has open issues
- Force a stuck implementer to retry without changing something

**If subagent asks questions:**

- Answer clearly and completely
- Provide additional context if needed
- Do not rush them into implementation

**If reviewer finds issues:**

- Implementer (same subagent) fixes them
- Reviewer reviews again
- Repeat until approved
- Do not skip the re-review

**If subagent fails a task:**

- Dispatch a fix subagent with specific instructions
- Do not try to fix manually (context pollution in the main thread)

## Integration

**Upstream (feeds into this skill):**

- **plan-phases** -- Produces the Phase Plan Document that this skill executes

**Skills used during execution:**

- **git-worktrees** -- REQUIRED: Create isolated workspace for the phase branch
- **tdd** -- Subagents follow TDD patterns during implementation
- **code-review** -- Template for final phase-wide code review
- **finish-branch** -- Finalize the phase branch after all chunks complete

**Downstream (this skill feeds into):**

- **review-phase** -- Reviews the completed phase branch as a whole
- **document-phase** -- Generates documentation for the phase
- **push-stacked-prs** -- Creates stacked PRs from the phase branch
