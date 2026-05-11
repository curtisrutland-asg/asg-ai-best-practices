# ASG AI Best-Practices Guide (research repo)

This repo is the working area for producing **per-repo `CLAUDE.md` templates**
that document Arizona Software Group's standard way of building applications.
The eventual readers are coding agents (Claude Code, etc.) and the humans who
work alongside them.

## What lives here

```
ai-best-practices/
  README.md          (this file)
  TAXONOMY.md        Master checklist of topics + status
  templates/         The deliverables — drop one of these into a real ASG repo
    CLAUDE.dotnet-backend.md
    CLAUDE.web-frontend.md
    CLAUDE.fullstack.md
  topics/            Raw research per topic — interview Q&A, rationale, examples
    project-structure.md
    naming-conventions.md
    testing.md
    ...
  decisions/         Architecture/decision log entries (ADRs)
    README.md
```

## Status

This is **Phase A: Bootstrap**. The scaffolding exists; topic content has
not yet been authored. See `TAXONOMY.md` for what's drafted vs. open.

## How to use this repo

**As a researcher (humans, or agents helping the researcher):**

1. Pick a `[ ] not-started` topic from `TAXONOMY.md`.
2. Open `topics/<topic>.md`. Each file has interview questions pre-baked at
   the top — answer them with the user.
3. Fill in the **Convention**, **Rationale**, and **Examples** sections.
4. Update status in `TAXONOMY.md` to `[~] drafted`, then `[x] confirmed` after
   user review.
5. When ~80% of topics are confirmed, assemble the `templates/CLAUDE.*.md`
   files from the topic content.

**As a consumer (a new ASG repo wanting a CLAUDE.md):**

1. Pick the template that matches your repo type (backend, frontend, full-stack).
2. Copy it to your repo's root as `CLAUDE.md`.
3. Override or extend any sections that need repo-specific detail. Don't delete
   sections without good reason — agents rely on consistent structure across
   ASG repos.

## Design principles for the output templates

- **Terse, imperative voice.** "Use FluentValidation for request DTOs" — not
  essay prose.
- **Concrete > abstract.** A 5-line code example beats a paragraph.
- **Anti-patterns named explicitly.** "Don't do X" is more reliable than
  "do Y" alone.
- **Layered.** Must-know rules at the top; deeper conventions later.
- **Under ~200 lines per template.** Anything longer goes into a linked
  reference file.

## Open questions

These need user input before Phase C (template assembly):

1. Which frontend framework(s) does ASG standardize on? (React / Angular /
   Blazor / mix)
2. Target .NET version (e.g., .NET 8 LTS)?
3. Existing internal docs to incorporate (Confluence / SharePoint / wiki)?
4. Will other ASG developers be interviewed, or is the user sole-source?

## Plan of record

`C:\Users\inser\.claude\plans\i-am-interested-in-cryptic-quiche.md`
(approved 2026-05-11).
