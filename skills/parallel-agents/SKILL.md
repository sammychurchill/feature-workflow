---
name: parallel-agents
description: >
  Use when there are 2+ independent tasks across different subsystems that
  can be solved without context from each other, such as unrelated test
  failures or independent lint errors across separate files.
---

# Parallel Agents

Dispatch two or more independent tasks to concurrent sub-agents using the
Agent tool. Each agent works in isolation on a focused problem.

---

## When to Use

Parallel agents are appropriate when ALL of these are true:

- There are **2+ independent tasks** that need attention.
- The tasks touch **different files or subsystems**.
- Solving one task does **not require output** from another.
- There is **no shared mutable state** between the problems.

### Good Candidates

| Scenario | Why Parallel Works |
|---|---|
| 3+ test failures across different modules | Each failure is self-contained |
| Multiple subsystems broken independently | No interaction between fixes |
| Independent lint/type errors in separate files | Fixes do not conflict |
| Writing tests for unrelated functions | No shared setup or state |

### When NOT to Use

| Scenario | Why Parallel Fails |
|---|---|
| Related failures (fix one, others may resolve) | Wasted work if root cause is shared |
| Need full context of the entire system | Single agent with full context is better |
| Exploratory debugging (unclear what is wrong) | Need to investigate before parallelizing |
| Shared state issues (race conditions, globals) | Agents may produce conflicting fixes |
| Cascading failures from one root cause | Fix the root cause first, then reassess |

---

## The Pattern

### Step 1: Identify Independent Domains

Group problems by what is broken, not by symptom:

- Read all error messages and stack traces.
- Identify which files and modules are involved in each problem.
- Verify there is no overlap between groups.
- If two problems touch the same file, they are NOT independent — handle
  them sequentially.

### Step 2: Create Focused Agent Tasks

Each agent task must be:

| Property | Requirement |
|---|---|
| **Scoped** | Clear boundary on what files/modules to touch |
| **Self-contained** | All necessary context included in the prompt |
| **Specific** | Exact goal stated, not open-ended |
| **Constrained** | Explicit limits on what NOT to do |
| **Measurable** | Clear definition of success (e.g., "tests pass") |

### Agent Prompt Structure

Every sub-agent prompt should follow this structure:

```
TASK: <one-sentence description of what to fix>

CONTEXT:
- Error message: <exact error or test failure>
- File(s) involved: <list of files>
- Relevant background: <any context the agent needs>

SCOPE:
- ONLY modify: <list of files the agent may touch>
- Do NOT modify: <explicit exclusions if needed>

GOAL:
- <specific, testable outcome>

VERIFICATION:
- Run: <exact test command to verify the fix>
- Expected: <what success looks like>
```

### Step 3: Dispatch in Parallel

Use the Agent tool to dispatch all sub-agents concurrently. Each agent
runs independently with no awareness of the others.

### Step 4: Review and Integrate

After all agents complete:

1. **Read all summaries** — Understand what each agent changed.
2. **Check for conflicts** — Did any agents modify the same file?
   If so, resolve manually.
3. **Run the full test suite** — Individual agent tests passing does
   not guarantee the combined changes work together.
4. **Commit coherently** — Each agent's fix should be a separate commit
   with a clear message.

---

## Example: Three Independent Test Failures

### Situation

Test suite reports 3 failures:
- `tests/auth/login.test.ts` — TypeError in password hashing
- `tests/api/orders.test.ts` — 404 on order creation endpoint
- `tests/email/templates.test.ts` — Missing template file reference

These are in separate modules with no shared code paths.

### Agent 1: Fix Auth Login Test

```
TASK: Fix TypeError in password hashing during login test.

CONTEXT:
- Error: TypeError: Cannot read property 'hash' of undefined
- File: src/auth/password.ts, line 23
- Test: tests/auth/login.test.ts

SCOPE:
- ONLY modify: src/auth/password.ts, tests/auth/login.test.ts
- Do NOT modify: any other auth files

GOAL:
- login.test.ts passes with no TypeError

VERIFICATION:
- Run: npx jest tests/auth/login.test.ts
- Expected: All tests pass
```

### Agent 2: Fix Orders API Test

```
TASK: Fix 404 error on order creation endpoint.

CONTEXT:
- Error: Expected 201 but received 404 for POST /api/orders
- File: src/api/routes/orders.ts
- Test: tests/api/orders.test.ts

SCOPE:
- ONLY modify: src/api/routes/orders.ts, tests/api/orders.test.ts
- Do NOT modify: any other route files

GOAL:
- POST /api/orders returns 201 with valid payload

VERIFICATION:
- Run: npx jest tests/api/orders.test.ts
- Expected: All tests pass
```

### Agent 3: Fix Email Template Test

```
TASK: Fix missing template file reference in email module.

CONTEXT:
- Error: ENOENT: no such file or directory 'templates/welcome.html'
- File: src/email/sender.ts, line 15
- Test: tests/email/templates.test.ts

SCOPE:
- ONLY modify: src/email/sender.ts, src/email/templates/, tests/email/templates.test.ts
- Do NOT modify: any other email files

GOAL:
- Template test passes with correct file reference

VERIFICATION:
- Run: npx jest tests/email/templates.test.ts
- Expected: All tests pass
```

### After All Agents Complete

```
1. Read summaries:
   - Agent 1: Fixed missing bcrypt import in password.ts
   - Agent 2: Fixed route registration — path was /orders not /api/orders
   - Agent 3: Updated template path to use __dirname-relative resolution

2. Check conflicts: No overlapping files. Clean.

3. Run full suite: npx jest --runInBand
   Result: 247 tests passed, 0 failed.

4. Commit each fix separately.
```

---

## Common Mistakes

| Mistake | Consequence | Correct Approach |
|---|---|---|
| Scope too broad ("fix all auth issues") | Agent wanders, makes unrelated changes | Scope to specific files and specific errors |
| No context in prompt | Agent wastes time re-discovering what you know | Include error messages, file paths, and background |
| No constraints on what to modify | Agent touches shared files, causes conflicts | Explicitly list allowed and disallowed files |
| Vague success criteria ("make it work") | Agent cannot verify its own fix | State the exact test command and expected output |
| Parallelizing related failures | Duplicate or conflicting fixes | Check for shared root causes first |
| Skipping full test suite after integration | Combined changes break something | Always run the full suite after merging agent work |

---

## Integration Points

- **Called by**: execute-phase (when multiple independent problems
  are identified), systematic-debugging (when failures are confirmed
  independent)
- **Pairs with**: code-review (review each agent's output),
  tdd (agents follow TDD within their scope)
