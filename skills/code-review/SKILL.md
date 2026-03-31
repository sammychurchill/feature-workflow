---
name: code-review
description: >
  Use when code changes are ready for review -- after chunk completion, after
  major features, or before any merge to a base branch. Dispatches a
  code-reviewer sub-agent to catch issues.
---

# Code Review

Dispatch a focused code-reviewer sub-agent to catch issues before they reach
the main branch. Reviews are mandatory at specific points and optional at
others.

---

## When to Request Review

### Mandatory

| Trigger | Why |
|---|---|
| After each chunk in phase execution | Catch issues while context is fresh |
| After a major feature is complete | Holistic review before integration |
| Before any merge to base branch | Last gate before code enters main |

### Optional (But Recommended)

| Trigger | Why |
|---|---|
| When stuck on a problem | Fresh eyes from a reviewer agent can spot what you missed |
| Before a significant refactor | Validate approach before investing effort |
| After a complex bug fix | Ensure the fix does not introduce new problems |

---

## How to Request a Review

### Step 1: Determine the Diff Boundaries

Get the git SHAs that bracket the work being reviewed:

```
BASE_SHA  — The commit hash from before the task started.
            Use `git merge-base HEAD <base-branch>` or the SHA recorded
            when the chunk began.

HEAD_SHA  — The current commit hash.
            Use `git rev-parse HEAD`.
```

### Step 2: Prepare Review Context

Gather these values before dispatching:

| Placeholder | What to Fill In |
|---|---|
| `WHAT_WAS_IMPLEMENTED` | One-sentence summary of the change |
| `PLAN_OR_REQUIREMENTS` | Paste the relevant chunk or phase from the plan |
| `BASE_SHA` | Commit hash before the work started |
| `HEAD_SHA` | Current commit hash after the work |
| `DESCRIPTION` | Any additional context the reviewer needs |

### Step 3: Dispatch the Code-Reviewer Sub-Agent

Use the Agent tool to dispatch a sub-agent. Load the prompt template from
`./reviewer-template.md` and fill in all placeholders listed above.

The sub-agent will:
1. Run `git diff <BASE_SHA>..<HEAD_SHA>` to see all changes.
2. Analyze the diff against the plan/requirements.
3. Categorize findings by severity.
4. Return a structured review report.

### Step 4: Act on Feedback

| Severity | Action | Can You Proceed? |
|---|---|---|
| **Critical** | Fix immediately. Do not continue until resolved. | No |
| **Important** | Fix before proceeding to the next chunk or phase. | No |
| **Minor** | Note for later. Fix in a cleanup pass or separate commit. | Yes |
| **Nitpick** | Optional. Apply if quick, skip if not. | Yes |

---

## Example Workflow

```
1. Complete chunk 2.1 of the execution plan.

2. Record SHAs:
   BASE_SHA = abc1234   (commit before chunk 2.1 started)
   HEAD_SHA = def5678   (current commit after chunk 2.1)

3. Dispatch reviewer sub-agent via Agent tool:
   - Load ./reviewer-template.md
   - Fill placeholders:
     WHAT_WAS_IMPLEMENTED = "Add user authentication middleware"
     PLAN_OR_REQUIREMENTS = <paste chunk 2.1 from plan>
     BASE_SHA = abc1234
     HEAD_SHA = def5678
     DESCRIPTION = "New Express middleware, touches auth and session modules"

4. Receive review report:
   - Critical: SQL injection in query builder — fix immediately
   - Important: Missing input validation on email field — fix before next chunk
   - Minor: Inconsistent variable naming in test file — note for later

5. Fix Critical and Important issues:
   - Patch SQL injection vulnerability, commit
   - Add email validation, commit

6. Proceed to chunk 2.2.
```

---

## Red Flags

These rationalizations must be overridden:

| Rationalization | Why It Is Wrong |
|---|---|
| "This change is too simple to review" | Simple changes still introduce bugs. Review anyway. |
| "I'll review it myself" | Self-review misses blind spots. Dispatch the sub-agent. |
| "The Critical issue is not really critical" | If the reviewer says Critical, treat it as Critical. Investigate fully before downgrading. |
| "I'll fix the Important issues later" | Important issues compound. Fix before proceeding. |
| "We're running low on time" | Skipping review costs more time in debugging later. |

---

## Integration Points

- **Called by**: execute-phase (after each chunk), finish-branch (before merge)
- **Pairs with**: receive-feedback (for handling review results),
  tdd (reviewer checks test coverage)
