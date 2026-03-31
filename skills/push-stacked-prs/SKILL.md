---
name: push-stacked-prs
description: >
  Use when a documented phase branch is ready to push. Creates stacked PRs
  with one PR per chunk, independently reviewable and merging bottom-up.
---

# Push Stacked PRs

Create stacked pull requests from a completed phase branch, with one PR per
chunk. This is stage 6 of the execution workflow -- the final stage for a
single phase.

**Announce at start:** "I'm using the push-stacked-prs skill to create stacked
PRs for this phase."

**Core principle:** Each chunk is independently reviewable as its own PR.
Stacking preserves the dependency order and enables bottom-up merging.

## When to Use

Use this skill after document-phase has committed documentation to the phase
branch. The phase branch should contain all chunk commits plus documentation
commits, with all tests passing and all reviews approved.

## Branch Structure

```
main
  └── feature/phase-1
        ├── feature/phase-1/chunk-1  →  PR 1 (chunk-1 → main)
        ├── feature/phase-1/chunk-2  →  PR 2 (chunk-2 → chunk-1)
        └── feature/phase-1/chunk-M  →  PR M (chunk-M → chunk-M-1)
```

Each PR contains only the diff for its chunk. Reviewers can review each PR
independently in order. Merging happens bottom-up: PR 1 merges first, then
PR 2 retargets to main (or the base branch), and so on.

## The Process

### Step 1: Identify Chunk Boundaries

From the phase branch commit history, identify the commits that belong to each
chunk:

1. Read the phase plan to get the ordered chunk list.
2. Map commits to chunks based on commit messages or the known commit sequence
   from execute-phase.
3. Identify the documentation commit(s) -- these go into the final chunk's PR
   or a separate documentation PR, depending on project convention.

### Step 2: Create Chunk Branches

For each chunk, create a branch that contains all commits up to and including
that chunk:

- `feature/phase-N/chunk-1` -- contains chunk 1 commits only
- `feature/phase-N/chunk-2` -- contains chunk 1 + chunk 2 commits
- `feature/phase-N/chunk-M` -- contains all commits (including documentation)

Each branch builds on the previous one, preserving the dependency chain.

### Step 3: Push Branches to Origin

Push all chunk branches to the remote:

```
git push origin feature/phase-N/chunk-1
git push origin feature/phase-N/chunk-2
...
git push origin feature/phase-N/chunk-M
```

### Step 4: Create Stacked PRs

Create one PR per chunk, each targeting the previous chunk's branch (or the
base branch for the first chunk):

| PR | Source Branch | Target Branch | Content |
|---|---|---|---|
| PR 1 | feature/phase-N/chunk-1 | main (or base) | Chunk 1 changes |
| PR 2 | feature/phase-N/chunk-2 | feature/phase-N/chunk-1 | Chunk 2 changes |
| PR 3 | feature/phase-N/chunk-3 | feature/phase-N/chunk-2 | Chunk 3 changes |
| PR M | feature/phase-N/chunk-M | feature/phase-N/chunk-M-1 | Chunk M + docs |

For each PR:

- **Title:** Use a clear, descriptive title indicating the chunk's purpose.
  Include the phase and chunk number for ordering context (e.g.,
  "[Phase 1 / Chunk 2] Add token validation middleware").
- **Body:** Include:
  - Summary of what this chunk implements
  - Acceptance criteria from the phase plan
  - Test coverage notes
  - Note that this is part of a stacked PR set, with ordering context
    (e.g., "PR 2 of 4 -- depends on PR 1, required by PR 3")
  - Link to related PRs in the stack

Use `gh pr create` to create each PR.

### Step 5: Verify PR Stack

After all PRs are created:

1. Verify each PR shows only its chunk's diff (not cumulative).
2. Verify PR targets are correct (each targets the previous chunk branch).
3. Verify all PRs are linked and ordered in the descriptions.
4. List all PR URLs for the user.

### Step 6: Present Summary

Present the complete PR stack to the user:

```
## Stacked PRs for [Phase Name]

| PR | Title | Target | URL |
|---|---|---|---|
| 1 | [title] | main | [url] |
| 2 | [title] | chunk-1 | [url] |
| M | [title] | chunk-M-1 | [url] |

### Merge Order
Merge bottom-up: PR 1 first, then retarget PR 2 to main, merge, and so on.

### Next Steps
[More phases remain / Feature complete]
```

### Step 7: Check for Remaining Phases

After pushing the stacked PRs, check whether more phases remain in the feature
plan:

- **More phases remain:** Announce which phase is next and invoke execute-phase
  with the next Phase Plan Document.
- **No more phases:** Announce that the feature is complete. All phases have
  been implemented, reviewed, documented, and pushed as stacked PRs.

## Merge Workflow

The team merges stacked PRs bottom-up:

1. Review and merge PR 1 (chunk-1 into main).
2. Retarget PR 2 from chunk-1 to main. Review and merge.
3. Retarget PR 3 from chunk-2 to main. Review and merge.
4. Continue until all PRs are merged.

Retargeting is necessary because once a base branch is merged and deleted, the
next PR needs a new target. Most Git hosting platforms handle this
automatically or with a single click.

## Red Flags

**Never:**

- Push directly to main or the base branch
- Create a single monolithic PR for the entire phase (defeats the purpose of
  chunking)
- Create PRs with incorrect target branches (breaks the stack)
- Skip the verification step (incorrect targets cause merge conflicts)
- Push if tests are failing on the phase branch
- Rewrite commit history after PRs are created (invalidates reviews)

**Watch for:**

- Documentation commits that should be attributed to the correct chunk
- Chunk branches that accidentally include commits from later chunks
- PR descriptions that are missing stack ordering context
- Merge conflicts between chunks (should not happen if chunks were implemented
  in order, but verify)

## Integration

**Upstream (feeds into this skill):**

- **document-phase** -- Produces the final phase branch with documentation
  committed

**Skills used during push:**

- **git-worktrees** -- May be needed if working across multiple branches

**Downstream (this skill feeds into):**

- **execute-phase** -- If more phases remain, loops back to execute the next
  phase plan
