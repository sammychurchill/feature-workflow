---
name: Test-Driven Development
description: >
  Use when implementing any feature or bugfix, before writing production code.
  Enforces the Red-Green-Refactor cycle -- no production code without a failing
  test first.
---

# Test-Driven Development

## The Iron Law

**NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST.**

There are no exceptions unless the user explicitly grants one. This is not a guideline. It is a hard constraint.

---

## The Red-Green-Refactor Cycle

### 1. RED -- Write One Minimal Failing Test

Write a single test that:

- Has a clear, descriptive name stating expected behavior
- Tests real behavior, not implementation details
- Tests exactly one thing
- Uses no mocks unless absolutely unavoidable (and justify why)

### 2. VERIFY RED (MANDATORY)

Run the test. Watch it fail. Confirm all three:

- [ ] The test **fails** (not errors -- a test error is not a valid RED)
- [ ] The failure is **expected** (the assertion you wrote is the one that fails)
- [ ] It fails **because the feature is missing**, not because of a typo or setup bug

If the test **passes**: you are testing existing behavior. Fix the test so it tests the new behavior you intend to add.

If the test **errors**: fix the error (import, syntax, setup). Re-run. You are not in RED until the test fails on its assertion.

### 3. GREEN -- Write the Simplest Code to Pass

Write the minimum production code that makes the failing test pass. Nothing more.

- Do not add features the test does not require.
- Do not refactor.
- Do not "improve" beyond what the test demands.
- Do not anticipate future tests.

### 4. VERIFY GREEN (MANDATORY)

Run the test suite. Confirm all three:

- [ ] The new test **passes**
- [ ] All other tests **still pass**
- [ ] Output is **pristine** (no warnings, no skipped tests, no unexpected output)

If the new test **fails**: fix the production code, not the test. The test defined the contract in step 1.

### 5. REFACTOR -- Improve Under Green

Only after GREEN is verified:

- Remove duplication
- Improve names
- Extract helpers or utilities
- Simplify logic

Rules during refactor:

- All tests must stay green after every change
- Do not add new behavior
- If a refactor breaks a test, undo and try a smaller refactor

### 6. REPEAT

Go back to step 1 with the next behavior.

---

## Good vs Bad Tests

### Good Test

```typescript
test("returns empty array when no users match the filter", () => {
  const users = [
    { name: "Alice", role: "admin" },
    { name: "Bob", role: "admin" },
  ];

  const result = filterUsers(users, { role: "viewer" });

  expect(result).toEqual([]);
});
```

Why it is good:

- Name states the expected behavior
- Arranges real data
- Calls the real function
- Asserts on output, not internals
- Tests one thing

### Bad Test

```typescript
test("filterUsers works", () => {
  const mockDb = jest.fn().mockReturnValue([{ name: "Alice" }]);
  const service = new UserService(mockDb);

  const result = service.filterUsers("viewer");

  expect(mockDb).toHaveBeenCalledWith("SELECT * FROM users WHERE role = ?", [
    "viewer",
  ]);
  expect(result.length).toBe(1);
});
```

Why it is bad:

- Vague name ("works" means nothing)
- Mocks the database, then asserts on the mock call (testing implementation)
- Asserts on `.length` instead of actual content
- Tests two things (the SQL call and the result count)
- Tightly coupled to implementation -- any refactor breaks it

---

## Rationalization Prevention

| Rationalization | Reality |
| --- | --- |
| "This is too simple to test." | Simple code breaks. The test takes 30 seconds to write. Write it. |
| "I'll write the tests after." | A test that passes immediately proves nothing about the code you just wrote. It only proves the test matches the code. |
| "Tests after achieve the same goals." | Tests-after answer "what does the code do?" Tests-first answer "what should the code do?" These are fundamentally different questions. |
| "I already tested this manually." | Manual ad-hoc testing is not systematic. You checked the happy path once. The test suite checks every path every time. |
| "Deleting X hours of work is wasteful." | Sunk cost fallacy. The code exists without a failing test. It cannot be trusted. Delete it and rebuild correctly. |
| "TDD will slow me down." | TDD is faster than debugging. You find problems in seconds instead of hours. The slowdown is an illusion caused by short-term thinking. |
| "I know this code is correct." | You don't. Write the test. If you are right, it takes 30 seconds. If you are wrong, you just saved yourself a production incident. |
| "I'll just keep this code as reference." | No. Code written before the test poisons the process. You will "adapt" instead of derive, and you will encode the same assumptions. Delete it. |
| "The test framework makes this hard." | The test framework is not the problem. If something is hard to test, that is a design signal. Fix the design. |
| "This is just a prototype." | Prototypes become production code. If it has behavior, it has tests. |

---

## Red Flags

Stop immediately if you observe any of these:

- Writing production code with no failing test on screen
- A "RED" test that passes
- A "RED" test that errors instead of failing
- Skipping VERIFY RED or VERIFY GREEN
- Writing more production code than the test demands
- Refactoring while a test is red
- Adding behavior during a refactor step
- Multiple tests written before any production code
- Mocking things you own
- Tests that assert on implementation details (method calls, internal state)
- A test name that does not describe behavior ("test1", "works", "handles edge case")

---

## Verification Checklist

Run this checklist before claiming a cycle is complete:

- [ ] Failing test was written before production code
- [ ] Test was observed to fail (RED verified)
- [ ] Failure was on the assertion, not a setup error
- [ ] Simplest possible code was written to pass
- [ ] New test passes (GREEN verified)
- [ ] All other tests still pass
- [ ] Refactoring did not add behavior
- [ ] All tests are still green after refactoring

---

## When Stuck

| Situation | Action |
| --- | --- |
| Don't know what test to write | Write the simplest behavior you can describe in one sentence. Test that. |
| Test is too big | Split it. One assertion. One behavior. |
| Can't make the test fail | You are testing existing behavior. Change the assertion to target new behavior. |
| Can't make the test pass simply | The test may be too large a step. Write a simpler intermediate test. |
| Everything is mocked | Back up. Test at a higher level with real collaborators. Mocks are a last resort. |
| Refactoring breaks tests | Undo the refactor. Make a smaller change. Keep tests green at every step. |
| Unclear requirements | Stop coding. Ask the user what the behavior should be. |
| Test passes but code feels wrong | Finish the cycle (stay green). Write a new test that exposes the wrongness. |

---

## Debugging Integration

When a test fails unexpectedly during development:

1. Do not guess at fixes.
2. Read the failure message completely.
3. Follow the systematic debugging process: investigate root cause before attempting any fix.
4. The failing test is your reproduction case -- do not delete it.

---

## Wrote Code Before a Test?

Delete it. All of it. Start over with a failing test.

- Do not keep it as "reference."
- Do not "adapt" it.
- Do not write a test that matches the code you already wrote.

The code was written without a contract. It cannot be trusted. Rebuild it with the discipline of Red-Green-Refactor.

---

**Violating the letter of the rules is violating the spirit of the rules.**
