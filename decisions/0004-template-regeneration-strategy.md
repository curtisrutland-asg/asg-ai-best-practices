# 0004 — Template regeneration strategy

**Status:** Open — conversation deferred
**Date:** 2026-05-11
**Deciders:** Curtis Rutland (ASG)

## Context

`topics/*.md` are the comprehensive research artifacts (Convention +
Rationale + Examples + Anti-patterns). `templates/CLAUDE.*.md` are the
terse, agent-facing summary documents that get dropped into real ASG
repos as their `CLAUDE.md`. Templates are not a mechanical concatenation
of topics — they involve editorial decisions about what survives into
the short version.

The maintenance question: **how do templates get regenerated / kept in
sync with topics as topics evolve?** The repo has no mechanism today;
templates are skeleton placeholders.

## Constraint

**Manual curation is rejected.** "A human reads the diff in a topic file
and updates the templates" defeats the agent-readable thesis of this
project. The maintenance loop must be at least as agent-driven as the
research loop. Solutions that boil down to "humans copy-paste sections"
are non-starters.

Specifically, the rejected approach was: maintain a `templates/MAPPING.md`
of topic → template references, plus `Source:` annotations in each
template section, plus a discipline of "when you edit a topic, check
the map and update affected templates." This is just procedurally
enforced copy-paste.

## Options considered (and their status)

### Option A (rejected): Manual curation with traceability links

See "Constraint" above. Rejected.

### Option B: Deterministic build script

A script extracts marked sections from topics, concatenates into
templates. Templates carry a "regenerated, do not edit" banner; CI
verifies they match. Loses the editorial layer — templates can only
emit what's structurally marked in topics.

- Still under consideration but not preferred — it forces topic files
  into a rigid structure to support a derived artifact, and it gives
  up the "agents make editorial judgments" benefit.

### Option C: AI-assisted regeneration via a Claude Code skill

A `/sync-templates` skill (or equivalent agent workflow) ingests topic
files and rewrites templates, preserving editorial curation. Run on
demand when topics have stabilized, or scheduled.

- **The most likely answer** given the project's premise. Specifics
  (prompt design, how reviewer approves the regenerated output, how to
  detect topic-change events, whether to run in CI or on demand) need
  to be designed.

### Option D (or hybrid): Other approaches not yet explored

Open to alternatives — e.g., a hybrid where a script handles purely
mechanical parts (gathering sections, length budgets) and an AI step
handles the editorial compression. Or templates that are themselves
just thin agents-readable manifests pointing to topic content. Worth
brainstorming when this conversation reopens.

## Decision

**Deferred** — conversation tabled for now. Captured here so the
constraint ("not manual") survives.

When this conversation reopens, the decision needs to specify:
1. **Which option** (B, C, hybrid, or other).
2. **The trigger** — what kicks off a regeneration (every topic edit?
   batch on demand? CI?).
3. **The review step** — how a human (or a second agent) confirms the
   regenerated output is correct before it lands.
4. **Failure modes** — what happens when a topic changes in a way the
   regeneration step can't handle.

## Consequences

- Phase C (template assembly) is **on hold** until this is decided.
  Building a one-off pilot of the .NET backend template by hand would
  produce work that needs to be re-done once the regeneration approach
  lands.
- Topic drafting (Phase B) can continue — topics are the source of
  truth and not affected by this decision.
- Anyone tempted to "just hand-write the template real quick" should
  stop and revisit this entry first.
