---
name: Systematic Debugging
description: >
  Use when encountering any bug, test failure, or unexpected behavior, before
  proposing fixes. Requires root cause investigation through four mandatory
  phases before any code changes.
---

# Systematic Debugging

## The Iron Law

**NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.**

Do not touch production code until you can state the root cause and explain the evidence that supports it. Guessing is not investigating. Changing things to see what happens is not investigating.

---

## Four Mandatory Phases

Complete each phase in order. Do not skip ahead.

### Phase 1: Root Cause Investigation

**Read error messages carefully.**

- Read the complete stack trace, not just the first line.
- Note exact line numbers, file names, and error types.
- Read the lines of code referenced in the trace.

**Reproduce consistently.**

- Write down the exact steps that trigger the bug.
- Confirm: does it happen every time? Only sometimes? Only with certain input?
- If you cannot reproduce it, you cannot fix it. Keep investigating.

**Check recent changes.**

- `git diff` -- what changed since it last worked?
- Recent commits -- did someone change a dependency, config, or shared module?
- Dependency updates -- did a library version change?

**Gather evidence in multi-component systems.**

- Add diagnostic instrumentation at each system boundary (API calls, database queries, message queues, service calls).
- Log data going in and data coming out at each boundary.
- Identify where the data goes wrong -- narrow the search space.

**Trace data flow.**

- Find the bad value or bad behavior.
- Where did it originate? Trace it backward through the code.
- Keep tracing upstream until you find the point where correct data becomes incorrect.

### Phase 2: Pattern Analysis

**Find working examples in the codebase.**

- Is there similar functionality that works correctly?
- What patterns do working examples follow?

**Compare against references completely.**

- Do not skim. Read the working example and the broken code side by side.
- Compare line by line if necessary.

**Identify ALL differences, however small.**

- Naming conventions, argument order, types, null checks, error handling, timing, async/await, whitespace in strings, encoding.
- The bug is often in a difference you initially dismissed as irrelevant.

**Understand dependencies.**

- What does this code depend on?
- Have any dependencies changed their contract, version, or behavior?
- Are there implicit dependencies (environment variables, file system state, network availability)?

### Phase 3: Hypothesis and Testing

**Form a single, specific hypothesis.**

State it explicitly: "I think [X] is the root cause because [Y evidence supports it]."

- The hypothesis must be specific enough to test.
- "Something is wrong with the database" is not a hypothesis.
- "The query returns stale data because the cache TTL is set to 24 hours and the data was updated 2 hours ago" is a hypothesis.

**Test minimally.**

- Change ONE variable at a time.
- Predict what will happen before you make the change.
- If the prediction is wrong, the hypothesis is wrong.

**Verify before continuing.**

- Did the minimal test confirm or refute the hypothesis?
- If confirmed: proceed to Phase 4.
- If refuted: do not add another fix on top. Form a NEW hypothesis and return to the start of Phase 3.

### Phase 4: Implementation

**Create a failing test case FIRST.**

- The test must reproduce the bug.
- Watch it fail. Confirm it fails for the reason you identified.

**Implement a single fix targeting the root cause.**

- Fix the root cause, not the symptom.
- One change. Not a "fix plus improvements."
- Not a "fix plus refactoring."

**Verify the fix.**

- The failing test now passes.
- All other tests still pass.
- The original reproduction steps no longer trigger the bug.

**If the fix does not work and you have tried fewer than 3 hypotheses:**

Return to Phase 1. Gather more evidence.

**If you have tried 3 or more hypotheses and none have worked:**

STOP. Three or more failed fix attempts is a strong signal of an architectural problem, not a simple bug. The assumptions you are operating under may be wrong. Discuss with the user before making more attempts.

---

## Red Flags

Stop immediately if you observe any of these:

- Changing code without being able to state the root cause
- Applying multiple fixes at once ("shotgun debugging")
- Fixing a symptom instead of the cause
- Skipping reproduction ("I think I know what it is")
- Ignoring parts of an error message or stack trace
- "Trying things" without a hypothesis
- Hypothesis has no supporting evidence
- Not verifying that a fix actually works
- Fixing the test instead of the code when a test fails unexpectedly
- Adding a workaround instead of fixing the root cause
- More than 3 failed fix attempts without stopping to reassess

---

## Rationalization Prevention

| Rationalization | Reality |
| --- | --- |
| "I know what this is, let me just fix it." | You don't. The last 5 times you said that, 3 of them were wrong. Investigate. |
| "It's probably this." | "Probably" is not evidence. Trace the data flow. |
| "Let me try a few things." | Undisciplined changes destroy evidence and waste time. One hypothesis, one test. |
| "The error message is misleading." | Error messages are rarely misleading. You are rarely reading them carefully enough. Read it again. |
| "This worked before, so the problem must be elsewhere." | "Before" is not now. Check what changed with `git diff`. |
| "I'll just add some error handling here." | That is a workaround, not a fix. Why is the error happening? |
| "The fix is obvious." | Then stating the root cause and evidence should take 10 seconds. State it. |
| "I don't have time to investigate properly." | You don't have time to debug the same issue three more times because you patched a symptom. |
| "Let me refactor while I'm in here." | No. Fix the bug. One change. Refactor separately after the fix is verified. |
| "Adding logging will help." | Adding logging is investigation, not fixing. Good. But do not call it a fix. |

---

## Quick Reference

| Step | Action | Output |
| --- | --- | --- |
| 1. Read | Full error message, stack trace, line numbers | Written notes on what the error actually says |
| 2. Reproduce | Exact steps, consistent result | Reproduction steps that trigger the bug every time |
| 3. Diff | `git diff`, recent commits, dependency changes | List of what changed since it last worked |
| 4. Trace | Follow data from symptom back to origin | The exact point where correct becomes incorrect |
| 5. Compare | Working example vs broken code, line by line | List of all differences |
| 6. Hypothesize | "X is the root cause because Y" | Single specific hypothesis with evidence |
| 7. Test | Change one variable, predict the outcome | Hypothesis confirmed or refuted |
| 8. Fix | Failing test, then single targeted code change | Bug reproduced in test, fix makes it pass |
| 9. Verify | All tests pass, reproduction steps no longer trigger bug | Evidence that the fix works |

---

## Supporting Techniques

**Binary search for the source of a bug:**

If you cannot trace the data flow directly, bisect. Find a known-good state and a known-bad state, then narrow the gap. `git bisect` works for commit-level bisection. For runtime bisection, add assertions at midpoints to narrow where data goes wrong.

**Rubber duck debugging:**

Explain the problem out loud (or in writing) step by step. State what you expect at each step and what actually happens. The act of precise articulation often reveals the gap in your understanding.

**Minimal reproduction:**

Strip away everything that is not necessary to trigger the bug. Remove unrelated code, simplify inputs, isolate the component. The smaller the reproduction, the easier the root cause is to see.

**Check assumptions explicitly:**

Log or assert every assumption. "I assume this value is not null" -- add an assertion. "I assume this function returns a string" -- log the type. Wrong assumptions are the most common root cause.

---

**Violating the letter of this process is violating the spirit of debugging.**
