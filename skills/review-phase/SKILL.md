---
name: review-phase
description: >
  Use when all chunks for a phase are committed and the phase branch needs
  holistic review. Runs full test suite, verifies cross-chunk integration,
  and validates phase-level acceptance criteria.
---

# Review Phase

Perform a comprehensive review of a completed phase branch. This is stage 4 of
the execution workflow -- after all chunks have been individually reviewed and
committed, this skill reviews the phase as a cohesive whole.

**Announce at start:** "I'm using the review-phase skill to review the
completed phase."

**Core principle:** Individual chunk reviews catch per-chunk issues. Phase
review catches integration issues, cross-chunk regressions, and validates that
the phase as a whole meets its acceptance criteria.

## When to Use

Use this skill after execute-phase has completed all chunks for a phase and
the finish-branch skill has finalized the phase branch. The phase branch should
have all chunk commits and be in a clean, passing state (per the final code
review in execute-phase).

## The Process

### Step 1: Run Full Test Suite

Run the complete test suite against the phase branch -- not just tests for
the changed code, but the entire project test suite.

- If tests **pass**: proceed to integration verification.
- If tests **fail**: investigate failures. Determine whether they are:
  - **Regressions from this phase**: must be fixed before proceeding. Loop
    back to execute-phase task loop to address the specific chunks.
  - **Pre-existing failures**: document them and proceed (they are not caused
    by this phase).

### Step 2: Integration Verification

Verify that all chunks work together as a cohesive whole:

- **Cross-chunk data flow**: Do outputs from earlier chunks feed correctly
  into later chunks? Are interfaces between chunks consistent?
- **Shared state**: If multiple chunks modify shared state (database schema,
  configuration, global state), verify they do not conflict.
- **Import/dependency chains**: Verify all cross-chunk imports resolve
  correctly and there are no circular dependencies introduced.
- **Edge cases at boundaries**: Test behavior at the boundaries where one
  chunk's work meets another's.

If integration issues are found, document them specifically (which chunks are
involved, what the conflict is, how to reproduce).

### Step 3: Review Complete Phase Diff

Review the cumulative diff for the entire phase -- not the individual chunk
diffs, but the full set of changes from the base branch to the phase branch
tip.

This perspective often reveals issues invisible at the chunk level:

- **Naming inconsistencies** across chunks (different authors may use
  different conventions)
- **Duplicate logic** that could be consolidated
- **Missing error handling** paths that span multiple chunks
- **Architectural drift** from the design spec when viewed as a whole
- **Dead code** introduced by one chunk and not cleaned up by later chunks

### Step 4: Validate Phase Acceptance Criteria

Check every phase-level acceptance criterion from the Phase Plan Document:

- Read each criterion and verify it is met by the implementation
- For each criterion, note: met, partially met, or not met
- If any criterion is partially met or not met, document specifically what
  is missing and which chunk should address it

Use the verify-completion skill patterns to ensure thoroughness.

### Step 5: Present Results to User

Present a structured review report:

```
## Phase Review: [Phase Name]

### Test Suite
- Total tests: [count]
- Passing: [count]
- Failing: [count] (pre-existing: [count], new: [count])

### Integration Verification
- [List of checks performed and results]
- Issues found: [list or "none"]

### Cumulative Diff Review
- Files changed: [count]
- Lines added: [count], removed: [count]
- Issues found: [list or "none"]

### Acceptance Criteria
| Criterion | Status | Notes |
|---|---|---|
| [criterion 1] | Met / Partial / Not Met | [details] |
| ... | ... | ... |

### Recommendation
[Approve / Issues to address / Abort]
```

### Step 6: Human Decision

**User decides:**

| Decision | Action |
|---|---|
| **Approved** | Proceed to document-phase |
| **Issues Found** | Loop back to execute-phase task loop to address specific chunks. Provide the user's feedback as context for the re-run. |
| **Abort** | Stop execution of this phase |

When looping back for issues, be specific about which chunks need rework and
what the issues are. Do not re-run the entire phase -- only the chunks that
need changes.

## Red Flags

**Never:**

- Skip the full test suite run (individual chunk tests are not sufficient)
- Review only the last chunk's diff instead of the cumulative phase diff
- Mark acceptance criteria as "met" without verifying the actual implementation
- Proceed to documentation with failing tests (unless pre-existing)
- Proceed with unresolved integration issues
- Rubber-stamp the review because individual chunks were already reviewed

**Watch for:**

- Tests that pass individually but fail when run together (ordering issues)
- Integration points that were not tested by any individual chunk
- Acceptance criteria that no single chunk fully owns (cross-cutting concerns)

## Integration

**Upstream (feeds into this skill):**

- **execute-phase** -- Produces the completed phase branch this skill reviews

**Skills used during review:**

- **verify-completion** -- Patterns for thorough acceptance criteria validation
- **systematic-debugging** -- If test failures need investigation

**Downstream (this skill feeds into):**

- **document-phase** -- Generates documentation for the approved phase
