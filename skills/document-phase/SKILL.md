---
name: document-phase
description: >
  Use when a phase has been reviewed and approved via review-phase and needs
  documentation. Updates changelog, API docs, README, and guides as applicable.
---

# Document Phase

Generate documentation for a completed phase's changes. This is stage 5 of the
execution workflow -- after the phase has been reviewed and approved via
review-phase.

**Announce at start:** "I'm using the document-phase skill to generate
documentation for this phase."

**Core principle:** Documentation is part of the deliverable, not an
afterthought. Every phase ships with updated docs that accurately reflect the
changes made.

## When to Use

Use this skill after review-phase has approved the completed phase branch.
The phase implementation is final -- documentation should describe what was
actually built, not what was planned.

## The Process

### Step 1: Analyze Phase Changes

Before writing any documentation, understand what changed:

1. **Read the phase diff** -- the cumulative changes from base to phase branch
   tip.
2. **Read the phase plan** -- understand the intended scope and objectives.
3. **Read the design spec** -- understand the broader feature context.
4. **Identify documentation-relevant changes:**
   - New public APIs, functions, classes, or modules
   - Changed behavior of existing APIs or features
   - New configuration options or environment variables
   - New CLI commands or flags
   - Changed installation or setup steps
   - New dependencies
   - Breaking changes
   - Migration steps required

### Step 2: Generate Changelog / Release Notes

Update the project's changelog or release notes file:

- Follow the existing changelog format in the project (if one exists)
- If no changelog exists, create entries that could be added later
- Include:
  - Summary of what was added, changed, fixed, or removed
  - Breaking changes (prominently noted)
  - Migration instructions if applicable
- Write for the end user, not the developer -- describe behavior changes, not
  implementation details
- Reference relevant issue numbers or PR numbers if available

### Step 3: Update API Documentation

If the phase introduced or modified public APIs:

- Update or create API reference documentation
- Document new endpoints, functions, classes, or modules
- Document changed signatures, parameters, return types
- Document new error codes or error conditions
- Include usage examples for new APIs
- Mark deprecated APIs if applicable
- Follow the existing API documentation format and conventions in the project

If no public API changes were made, skip this step and note it in the review
summary.

### Step 4: Update README and Guides

If the phase changes affect user-facing behavior:

- Update the README if it references changed features or setup steps
- Update getting-started guides if installation or setup changed
- Update usage guides if workflows or commands changed
- Update configuration documentation if new options were added
- Add new guide sections if the phase introduces entirely new capabilities

If no user-facing behavior changed, skip this step and note it in the review
summary.

### Step 5: Consistency Check

Before presenting to the user, verify documentation consistency:

- Do code examples in docs actually work with the current implementation?
- Do API signatures in docs match the actual code?
- Are there references to old behavior that should be updated?
- Are there broken links or references to renamed files/functions?
- Is the tone and style consistent with existing project documentation?

### Step 6: Present to User

Present the documentation changes for review:

```
## Documentation for [Phase Name]

### Changes Made
- [List of documentation files created or modified]

### Changelog Entry
[Show the changelog content]

### API Documentation
[Show new or updated API docs, or note "No API changes in this phase"]

### README / Guide Updates
[Show updates, or note "No user-facing behavior changes in this phase"]

### Skipped (Not Applicable)
- [List any steps skipped and why]
```

### Step 7: Human Decision

**User decides:**

| Decision | Action |
|---|---|
| **Approved** | Commit documentation changes to the phase branch |
| **Revise** | Regenerate documentation based on user feedback. Incorporate specific revision requests and re-present. |

After approval, commit the documentation to the phase branch as a separate
commit (not squashed into implementation commits). This keeps the documentation
changes reviewable independently.

## Guidelines

**Write for the audience:**

- Changelog entries: written for end users and downstream consumers
- API docs: written for developers integrating with the API
- README/guides: written for users setting up or using the project
- Do not write internal implementation notes as user documentation

**Follow existing conventions:**

- Match the project's existing documentation style, format, and tone
- Use the same heading structure, code block style, and example format
- If the project uses a documentation generator (JSDoc, Sphinx, rustdoc, etc.),
  write docs compatible with that tool

**Be accurate:**

- Document what was built, not what was planned
- Verify code examples against the actual implementation
- Do not document features that were deferred or cut during implementation

## Red Flags

**Never:**

- Skip documentation because "the code is self-documenting"
- Document planned features that were not actually implemented
- Write documentation that contradicts the actual implementation
- Commit documentation without user review
- Modify implementation code during the documentation phase
- Generate placeholder documentation ("TODO: add docs here")

**Watch for:**

- Phase changes that affect existing documentation elsewhere in the project
- New error conditions that should be documented for users
- Configuration changes that affect deployment or operations guides
- Breaking changes that need migration documentation

## Integration

**Upstream (feeds into this skill):**

- **review-phase** -- Approves the phase, signaling readiness for documentation

**Downstream (this skill feeds into):**

- **push-stacked-prs** -- Pushes the phase branch (including documentation
  commits) as stacked PRs
