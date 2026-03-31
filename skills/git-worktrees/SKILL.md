---
name: git-worktrees
description: >
  Use when you need an isolated git worktree for parallel or independent
  development work. Handles directory selection, gitignore enforcement, project
  setup, and baseline test verification.
---

# Git Worktrees

Create isolated git worktrees for parallel or independent development work.
Every worktree gets a verified clean baseline before any work begins.

---

## Directory Selection Priority

Determine where to place worktrees using this priority order:

| Priority | Check | Action |
|---|---|---|
| 1st | Existing `.worktrees/` directory | Use it (preferred location) |
| 2nd | Existing `worktrees/` directory | Use it |
| 3rd | CLAUDE.md specifies a worktree directory | Use the specified path |
| 4th | None of the above | Ask the user where to place worktrees |

---

## Safety Verification (Project-Local Directories Only)

When using a directory inside the project (`.worktrees/` or `worktrees/`),
the directory MUST be ignored by git. Committed worktree directories corrupt
the repository.

### Verification Steps

1. **Check if the directory is git-ignored**:
   ```
   git check-ignore -q .worktrees
   ```
   - Exit code 0 = ignored (safe, proceed).
   - Exit code 1 = NOT ignored (unsafe, fix first).

2. **If NOT ignored**: Add to `.gitignore`, commit the change, then proceed.
   ```
   # Add to .gitignore
   echo ".worktrees/" >> .gitignore

   # Commit the gitignore update
   git add .gitignore
   git commit -m "Add .worktrees to gitignore"
   ```

3. **Never create a worktree in an unignored project-local directory.**

This step does NOT apply to directories outside the project root (e.g.,
`/tmp/worktrees/` or `~/worktrees/`).

---

## Creation Steps

### Step 1: Detect Project Name

Read the project name from the git root directory name:
```
basename $(git rev-parse --show-toplevel)
```

### Step 2: Create the Worktree

```
git worktree add <WORKTREE_PATH> -b <BRANCH_NAME>
```

Where:
- `WORKTREE_PATH` = `<worktree-dir>/<branch-name>` (e.g.,
  `.worktrees/feat-auth-middleware`)
- `BRANCH_NAME` = descriptive branch name for the work (e.g.,
  `feat/auth-middleware` or `phase-2-chunk-1`)

### Step 3: Run Project Setup

Auto-detect the project type and run the appropriate setup command:

| Indicator File | Setup Command |
|---|---|
| `package-lock.json` | `npm ci` |
| `package.json` (no lock) | `npm install` |
| `yarn.lock` | `yarn install --frozen-lockfile` |
| `pnpm-lock.yaml` | `pnpm install --frozen-lockfile` |
| `Cargo.toml` | `cargo build` |
| `requirements.txt` | `pip install -r requirements.txt` |
| `pyproject.toml` | `pip install -e .` or project-specific |
| `go.mod` | `go mod download` |
| `Gemfile` | `bundle install` |

Run the setup command from within the worktree directory.

### Step 4: Verify Clean Baseline

Run the project's test suite from the worktree directory. The tests MUST
pass before any work begins.

- If tests **pass**: Record the baseline. Proceed.
- If tests **fail**: STOP. Report the failures to the user. Do NOT
  proceed with work in a worktree that has a failing baseline. Ask the
  user how to handle it.

### Step 5: Report

Report to the user:
- Worktree location (absolute path)
- Branch name
- Setup status (success/failure)
- Baseline test status (pass/fail with details if failed)

---

## Quick Reference

| Task | Command |
|---|---|
| List worktrees | `git worktree list` |
| Create worktree | `git worktree add <path> -b <branch>` |
| Remove worktree | `git worktree remove <path>` |
| Prune stale entries | `git worktree prune` |

---

## Example Workflow

```
1. User requests work on authentication middleware.

2. Check for worktree directory:
   - Found: .worktrees/ exists
   - Verify: git check-ignore -q .worktrees  ->  exit 0 (safe)

3. Create worktree:
   git worktree add .worktrees/feat-auth-middleware -b feat/auth-middleware

4. Detect project type:
   - Found: package-lock.json
   - Run: npm ci (in .worktrees/feat-auth-middleware/)

5. Verify baseline:
   - Run: npm test (in .worktrees/feat-auth-middleware/)
   - Result: 142 tests passed, 0 failed

6. Report:
   Worktree created at /home/user/project/.worktrees/feat-auth-middleware
   Branch: feat/auth-middleware
   Setup: npm ci completed successfully
   Baseline: 142 tests passing
```

---

## Common Mistakes

| Mistake | Consequence | Correct Approach |
|---|---|---|
| Creating worktree without checking gitignore | Worktree contents get committed | Always run `git check-ignore` first |
| Skipping baseline tests | Start work on a broken foundation | Always run tests before starting work |
| Using the same branch name as an existing branch | Git error; worktree creation fails | Check `git branch --list` first |
| Forgetting to run project setup | Missing dependencies cause false test failures | Always run setup before testing |
| Hardcoding the worktree directory | Breaks in projects with different conventions | Follow the directory selection priority |

---

## Red Flags

| Rationalization | Why It Is Wrong |
|---|---|
| "The directory is probably already ignored" | Verify with `git check-ignore`. Never assume. |
| "Tests take too long, skip the baseline" | A failing baseline means every subsequent failure is ambiguous. |
| "Tests are failing but it's probably unrelated" | Do not proceed. Ask the user. |
| "I know where worktrees go in this project" | Follow the directory selection priority. Check, do not assume. |

---

## Integration Points

- **Called by**: execute-phase (to create isolated work environments)
- **Pairs with**: finish-branch (cleans up the worktree after work
  is complete)
