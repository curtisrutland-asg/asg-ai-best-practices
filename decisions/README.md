# Decisions log

Architecture / convention decisions worth recording for posterity. Used when:

- A topic interview surfaces a real choice ASG made between alternatives,
  and the *why* is worth preserving so future agents (and humans) don't
  re-litigate it.
- ASG hasn't actually decided on a question yet — log it here as `Status: Open`
  so agents reading the topic file know the area is unstandardized.

## File naming

`NNNN-short-slug.md`, four-digit zero-padded, e.g.:

- `0001-monorepo-vs-polyrepo.md`
- `0002-frontend-framework-choice.md`

## Template per entry

```markdown
# NNNN — <title>

**Status:** Proposed | Accepted | Open | Superseded by NNNN
**Date:** YYYY-MM-DD
**Deciders:** <names>

## Context
What forced the question?

## Options considered
1. Option A — pros / cons
2. Option B — pros / cons

## Decision
What we chose.

## Consequences
What this means for ASG repos and agents working in them.
```

Keep entries short — a decision log is not a design doc.
