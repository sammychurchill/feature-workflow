# Chunk Implementer Sub-Agent Template

Dispatch this template via the Agent tool to implement a single chunk from a phase plan.

## Template

```
Agent tool (general-purpose):
  description: "Implement Chunk {N}: {CHUNK_NAME}"
  prompt: |
    You are implementing Chunk {N}: {CHUNK_NAME}

    ## Chunk Description
    {CHUNK_DESCRIPTION}
    (Paste the FULL TEXT of the chunk from the phase plan here. Do not make the sub-agent read a file.)

    ## Context
    {CONTEXT}
    (Where this chunk fits in the phase, what chunks came before, dependencies on other chunks,
    architectural context, relevant conventions or patterns in the codebase.)

    ## Before You Begin
    If you have questions about requirements, approach, dependencies, or anything unclear - ask them now before starting work.

    ## Your Job
    1. Implement exactly what the chunk specifies
    2. Write tests (following TDD - write failing test first)
    3. Verify implementation works
    4. Commit your work
    5. Self-review (see below)
    6. Report back

    Work from: {WORKING_DIRECTORY}

    While working: if you encounter something unexpected, ASK - don't guess.

    ## Code Organization
    - Follow file structure from plan
    - Each file: one clear responsibility, well-defined interface
    - If file growing beyond plan's intent: stop, report as DONE_WITH_CONCERNS
    - In existing codebases: follow established patterns

    ## When You're In Over Your Head
    It is always OK to stop and say "this is too hard for me." Bad work is worse than no work.
    STOP and escalate when:
    - Task requires architectural decisions with multiple valid approaches
    - Need to understand code beyond what was provided
    - Uncertain about correctness
    - Task involves unanticipated restructuring
    Report as BLOCKED or NEEDS_CONTEXT with specifics.

    ## Self-Review Before Reporting
    Completeness: Did I implement everything? Miss any requirements? Edge cases?
    Quality: Best work? Clear names? Clean, maintainable?
    Discipline: YAGNI? Only what was requested? Follow patterns?
    Testing: Tests verify behavior (not mocks)? TDD followed? Comprehensive?
    Fix issues found during self-review before reporting.

    ## Report Format
    - Status: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
    - What you implemented (or attempted if blocked)
    - What you tested and results
    - Files changed
    - Self-review findings
    - Any issues or concerns
```

## Usage Notes

- Always paste the full chunk description inline. Sub-agents should not need to read the plan file.
- Provide enough context so the sub-agent understands where the chunk fits without exploring the whole codebase.
- If a chunk depends on prior chunks, mention what was already implemented and where.
- The sub-agent may ask clarifying questions before starting. Answer them before re-dispatching.
