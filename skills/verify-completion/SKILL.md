---
name: Verify Completion
description: >
  Use when about to claim work is complete, fixed, or passing, before
  committing or creating PRs. No completion claims without fresh verification
  evidence from commands you just ran.
---

# Verify Completion

## The Iron Law

**NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE.**

"I believe it works" is not evidence. "It should pass" is not evidence. "I just ran it a few minutes ago" is not evidence. The only evidence is output from a command you just ran, right now, that you read in full.

---

## The Gate Function

Every completion claim must pass through this gate. No exceptions.

### 1. IDENTIFY

What command proves this claim? Be specific.

- "Tests pass" requires the actual test command for this project.
- "Build succeeds" requires the actual build command for this project.
- "Linter clean" requires the actual lint command for this project.
- "Bug fixed" requires reproducing the original symptom and showing it no longer occurs.

### 2. RUN

Execute the full command. Fresh. Right now.

- Not a cached result.
- Not a partial run.
- Not a previous run from earlier in the session.
- The complete command, not a subset.

### 3. READ

Read the full output.

- Check the exit code.
- Count failures, errors, and warnings.
- Read every line, not just the summary.
- Look for skipped tests, partial results, or suppressed warnings.

### 4. VERIFY

Does the output actually confirm the claim?

- **NO**: State the actual status with evidence. "Tests: 3 failed out of 47. Failures: [list them]."
- **YES**: State the claim with evidence. "Tests: 47 passed, 0 failed, exit code 0."

### 5. ONLY THEN

Make the claim. With the evidence attached.

---

## Common Failures

| Claim | Requires | Not Sufficient |
| --- | --- | --- |
| Tests pass | Test command output showing 0 failures, exit code 0 | A previous run from earlier. "Should pass." Partial test run. |
| Linter clean | Linter output showing 0 errors and 0 warnings, exit code 0 | Partial check. Running linter on one file when the claim is about the project. |
| Build succeeds | Build command output with exit code 0 | Linter passing (linter is not the build). Previous build output. |
| Bug fixed | Original symptom no longer reproducible, test proves it | "I changed the code." Code change is not evidence of fix. |
| Feature complete | All acceptance criteria verified with evidence | "I implemented it." Implementation is not completion. |
| Agent completed task | VCS diff shows the expected changes, tests pass | Agent reporting "Done" or "Success." Agent claims are not evidence. |
| No regressions | Full test suite passing after changes | Subset of tests. "I only changed X so Y can't be affected." |

---

## Red Flags

Stop and re-verify if you catch yourself doing any of these:

- Using the word **"should"** in a completion claim ("tests should pass")
- Using the word **"probably"** ("this probably works now")
- Using the phrase **"seems to"** ("seems to be working")
- Expressing satisfaction or confidence before running verification
- Trusting an agent's self-reported success without checking its output
- Saying **"just this once"** about skipping verification
- Referring to a previous run instead of a fresh one
- Claiming completion of something you have not explicitly tested
- Skipping verification because "the change was small"
- Assuming passing a linter means the build works
- Assuming the build working means the tests pass

---

## Rationalization Prevention

| Rationalization | Reality |
| --- | --- |
| "The change was trivial, no need to verify." | Trivial changes cause production outages. Verify. |
| "I just ran it a minute ago." | A minute ago is not now. You may have changed something since. Run it again. |
| "The CI will catch it." | CI is a safety net, not a substitute for local verification. You are claiming completion now, not after CI runs. |
| "Only one file changed, so nothing else could break." | You do not know the dependency graph by heart. Run the full suite. |
| "The agent said it succeeded." | Agents hallucinate success. Check the diff. Run the tests. |
| "I'll verify after I commit." | Then you are not claiming completion. You are claiming "probably done, will check later." Say that instead. |
| "Tests are slow, I'll just run the relevant ones." | Then your claim is "relevant tests pass," not "tests pass." Be precise about what you verified. |
| "It compiled, so it works." | Compilation checks types and syntax, not behavior. Run the tests. |
| "I'm confident in this code." | Confidence is not evidence. Run the command. |

---

## Key Patterns

### Tests Pass (Correct)

```
1. Run: npm test
2. Read output: "47 passed, 0 failed, 0 skipped"
3. Exit code: 0
4. Claim: "All 47 tests pass."
```

### Tests Pass (Incorrect)

```
1. Changed the code.
2. "Tests should pass now."
```

This is not verification. No command was run. No output was read.

---

### Regression Test -- Red-Green (Correct)

```
1. Write test for the bug.
2. Run: npm test
3. Read output: new test FAILS (confirms it catches the bug).
4. Fix the code.
5. Run: npm test
6. Read output: "48 passed, 0 failed, 0 skipped"
7. Exit code: 0
8. Claim: "Bug is fixed. Regression test confirms: fails before fix, passes after."
```

### Regression Test (Incorrect)

```
1. Write test for the bug.
2. Fix the code.
3. Run: npm test
4. Read output: "48 passed, 0 failed"
5. Claim: "Bug is fixed."
```

The test was never observed to fail. It may not actually test the bug. The red-green cycle was skipped.

---

### Build (Correct)

```
1. Run: npm run build
2. Read output: "Build completed successfully"
3. Exit code: 0
4. Claim: "Build succeeds."
```

### Build (Incorrect)

```
1. Run: npm run lint
2. Read output: "0 errors"
3. Claim: "Build succeeds."
```

Linting is not building. The claim does not match the evidence.

---

### Requirements (Correct)

```
1. Requirement: "Users can reset their password via email."
2. Run test for password reset flow.
3. Read output: test passes.
4. Manually verify: trigger reset, receive email, follow link, set new password, log in.
5. Claim: "Password reset feature complete. Automated test passes. Manual verification confirms full flow."
```

### Requirements (Incorrect)

```
1. Requirement: "Users can reset their password via email."
2. Wrote the code.
3. Claim: "Password reset feature complete."
```

Writing code is not completing a requirement. Where is the evidence that it works?

---

### Agent Delegation (Correct)

```
1. Delegated task to agent.
2. Agent reports: "Done."
3. Run: git diff
4. Read output: changes match expected scope.
5. Run: npm test
6. Read output: "52 passed, 0 failed"
7. Claim: "Agent completed the task. Diff confirms expected changes. All 52 tests pass."
```

### Agent Delegation (Incorrect)

```
1. Delegated task to agent.
2. Agent reports: "Done. All tests pass."
3. Claim: "Task complete."
```

Agent self-reports are not evidence. The agent may have hallucinated, run a partial suite, or misunderstood the task. Verify independently.

---

**Violating the letter of this rule is violating the spirit of this rule.**
