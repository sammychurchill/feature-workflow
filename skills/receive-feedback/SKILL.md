---
name: receive-feedback
description: >
  Use when code review feedback or reviewer comments have been received and
  need to be addressed. Processes each item with technical rigor -- no
  performative agreement, only verified responses.
---

# Receive Feedback

Process code review feedback with technical discipline. Every piece of
feedback gets the same rigorous treatment: read it, verify it against the
codebase, evaluate whether it applies, then respond with substance.

---

## Response Pattern

For EACH feedback item, follow these steps in order:

### 1. READ

Read the complete feedback. Do not skim. Do not start implementing after
reading the first item.

### 2. UNDERSTAND

Restate the requirement in your own words. Not a copy-paste — demonstrate
that you understand what the reviewer is actually asking for and why.

### 3. VERIFY

Check the feedback against the actual codebase:
- Does the code the reviewer references actually exist as described?
- Is the reviewer looking at the current version?
- Does the issue reproduce?

### 4. EVALUATE

Is this feedback technically sound for THIS codebase?
- Does it account for existing patterns and conventions?
- Does it consider the actual usage context?
- Is the suggested fix compatible with the rest of the system?

### 5. RESPOND

Two valid responses:

**Technical acknowledgment** (when feedback is correct):
- State what the issue is.
- State what you will change and why.
- No filler. No gratitude.

**Reasoned pushback** (when feedback is incorrect or inapplicable):
- State what the reviewer suggested.
- State why it does not apply, with evidence from the codebase.
- Propose an alternative if one exists.

### 6. IMPLEMENT

Fix one item at a time. Run tests after each fix. Do not batch fixes —
if one fix breaks something, you need to know which one.

---

## Forbidden Phrases

Never use any of these. They add no information and waste tokens:

| Forbidden | Why |
|---|---|
| "You're absolutely right!" | Performative. Just fix it. |
| "Great point!" | Performative. Just fix it. |
| "Thanks for catching that!" | Performative. Just fix it. |
| "Good catch!" | Performative. Just fix it. |
| "I appreciate the feedback" | Performative. Just fix it. |
| "That's a great suggestion" | Performative. Just fix it. |
| Any gratitude expression | Actions speak. Fix it or push back. |

Instead: state the issue, state the fix, implement the fix.

---

## Handling Unclear Feedback

When feedback is ambiguous, incomplete, or contradictory:

1. **STOP** — Do not implement anything.
2. **Collect** — Identify ALL unclear items across the entire review.
3. **ASK** — Ask for clarification on all unclear items at once.
   Do not ask one at a time; batch your questions.
4. **WAIT** — Do not guess at intent. Wait for answers.

Implementing unclear feedback leads to rework. Asking upfront saves time.

---

## YAGNI Check

When a reviewer suggests "implementing this properly" or "doing it the
right way":

1. Grep for actual usage of the code in question.
2. Check how many callers exist.
3. Check if the "proper" implementation serves any current use case.
4. If the only benefit is hypothetical future flexibility — push back.

The reviewer may be right. But verify with evidence, not vibes.

---

## When to Push Back

Push back is not defiance — it is engineering rigor. Push back when:

| Situation | How to Push Back |
|---|---|
| Fix would break existing functionality | Show the specific code that would break and explain why |
| Reviewer lacks context about a constraint | Provide the constraint with a reference to where it is documented or enforced |
| Suggestion violates YAGNI | Show that no current code path needs the suggested abstraction |
| Feedback is technically incorrect | Provide a counterexample or reference to documentation |
| Legacy reasons prevent the change | Explain the legacy constraint and what it would take to remove it |
| Conflicts with an architectural decision | Reference the design spec or decision record |

---

## Implementation Order

When multiple feedback items need fixing:

| Priority | Category | Rationale |
|---|---|---|
| 1st | Blocking issues | Tests fail, build breaks, security holes |
| 2nd | Simple fixes | Naming, formatting, small logic tweaks — fast wins |
| 3rd | Complex changes | Refactors, redesigns, multi-file changes |

Within each category, fix items in the order they appear in the codebase
(top-down through files), not the order the reviewer listed them.

---

## Common Mistakes

| Mistake | Consequence | Correct Approach |
|---|---|---|
| Implementing feedback without verifying | Introduce bugs from incorrect review | VERIFY against codebase first |
| Batching all fixes into one commit | Cannot isolate which fix caused a failure | One fix, one test run, one commit |
| Agreeing with everything to avoid conflict | Ship worse code to be agreeable | Push back with evidence when warranted |
| Implementing unclear feedback by guessing | Rework when guess is wrong | ASK for clarification before starting |
| Skipping the EVALUATE step | Apply changes that do not fit the codebase | Check compatibility before implementing |
| Fixing items in reviewer's listed order | Miss dependencies between fixes | Fix by priority, then by file order |

---

## Examples

### Good Response

> **Reviewer**: "The `processOrder` function should validate the input
> schema before processing."
>
> **Response**: "The `processOrder` function at line 42 of `orders.ts`
> currently trusts the caller to provide valid input. Adding Zod validation
> at the entry point will catch malformed orders before they hit the
> database layer. Implementing now."

### Bad Response

> **Reviewer**: "The `processOrder` function should validate the input
> schema before processing."
>
> **Response**: "Great catch! You're absolutely right that we should
> validate the input. Thanks for pointing this out! I'll add validation
> right away."

The bad response says nothing about what will change, where, or why.
The good response demonstrates understanding and states the plan.

---

## Integration Points

- **Called by**: code-review (when review results arrive),
  execute-phase (when chunk review returns feedback)
- **Pairs with**: code-review (processes its output), tdd (each fix
  gets tested)
