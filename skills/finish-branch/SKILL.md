---
name: finish-branch
description: >
  Use when development work on a feature branch is done and you need to wrap
  up -- runs test verification, presents structured options (merge, push PR,
  keep, or discard), and handles worktree cleanup.
---

# Finish Branch

Complete development work with test verification, a structured set of
options, and proper cleanup. No work leaves this skill without passing
tests.

---

## Process

### Step 1: Verify Tests

Run the full test suite. This is non-negotiable.

- If tests **pass**: Proceed to Step 2.
- If tests **fail**: STOP. Show the failures. You CANNOT proceed until
  all tests pass. Fix the failures or ask the user for guidance.

### Step 2: Determine Base Branch

Detect the base branch that this feature branch diverged from:

```
git merge-base --fork-point main HEAD
```

If `main` does not work, try `master`, `develop`, or use:
```
git log --oneline --graph --all
```
to identify the correct base branch. Confirm with the user if ambiguous.

### Step 3: Present Options

Present exactly these four options. Do not add, remove, or reorder them.

> **All tests pass. How would you like to finish this branch?**
>
> 1. **Merge back to `<base-branch>` locally**
>    Merge this branch into `<base-branch>` on your machine.
>
> 2. **Push and create a Pull Request**
>    Push this branch to the remote and open a PR against `<base-branch>`.
>
> 3. **Keep the branch as-is**
>    Leave everything in place. No merge, no push, no cleanup.
>
> 4. **Discard this work**
>    Delete the branch and all changes. This is irreversible.

Wait for the user to choose. Do not assume a default.

### Step 4: Execute Chosen Option

#### Option 1: Merge Locally

```
git checkout <base-branch>
git merge <feature-branch> --no-ff
```

Use `--no-ff` to preserve the branch history in the merge commit.

After merge, proceed to Step 5 (cleanup).

#### Option 2: Push and Create PR

```
git push -u origin <feature-branch>
```

Then create the PR:
```
gh pr create --base <base-branch> --title "<PR title>" --body "<PR body>"
```

Ask the user for the PR title and description, or generate from the
commit history if the user prefers.

After push and PR creation, proceed to Step 5 (cleanup).

#### Option 3: Keep As-Is

Do nothing. Report the branch name and location. Skip Step 5 entirely.
The worktree and branch remain in place for future work.

#### Option 4: Discard

This option requires explicit confirmation.

> **This will permanently delete the branch `<feature-branch>` and all
> its changes. Type "discard" to confirm.**

- If user types "discard": Delete the branch and proceed to Step 5.
  ```
  git checkout <base-branch>
  git branch -D <feature-branch>
  ```
- If user types anything else: Cancel. Return to Step 3.

### Step 5: Cleanup Worktree

**Applies to options 1, 2, and 4 only.** NOT option 3.

If the work was done in a git worktree (created by the git-worktrees
skill):

```
git worktree remove <worktree-path>
git worktree prune
```

Verify the worktree was removed:
```
git worktree list
```

If the work was NOT in a worktree, skip this step.

---

## Quick Reference

| Option | Merge? | Push? | Cleanup Worktree? |
|---|---|---|---|
| 1. Merge locally | Yes | No | Yes |
| 2. Push and PR | No | Yes | Yes |
| 3. Keep as-is | No | No | No |
| 4. Discard | No | No | Yes |

---

## Common Mistakes

| Mistake | Consequence | Correct Approach |
|---|---|---|
| Proceeding with failing tests | Broken code reaches base branch | STOP. Fix tests first. No exceptions. |
| Assuming the base branch is `main` | Merge into wrong branch | Detect with `git merge-base` and confirm |
| Skipping `--no-ff` on local merge | Lose branch history | Always use `--no-ff` for feature merges |
| Discarding without confirmation | Irreversible data loss | Require typed "discard" confirmation |
| Cleaning up worktree for option 3 | User loses their workspace | Only clean up for options 1, 2, and 4 |
| Forgetting `git worktree prune` | Stale worktree references remain | Always prune after removing a worktree |

---

## Red Flags

| Rationalization | Why It Is Wrong |
|---|---|
| "Most tests pass, close enough" | All tests must pass. No exceptions. |
| "I'll fix the tests after merging" | Broken tests on the base branch block everyone. |
| "The user probably wants to merge" | Present all 4 options. Let the user decide. |
| "Discard confirmation is overkill" | Irreversible operations always require confirmation. |
| "The worktree will clean itself up" | Worktrees persist until explicitly removed. |

---

## Integration Points

- **Called by**: execute-phase (after all chunks in a phase complete)
- **Pairs with**: git-worktrees (cleans up the worktree created by
  that skill), code-review (review should happen before reaching
  this skill)
