# CLAUDE.md — ASG full-stack repo (backend + web frontend in one repo)

> **Skeleton.** This template is not yet populated; topic content is still
> being captured in `../topics/`. Full-stack template combines the backend
> and frontend templates, with extra sections for cross-tier concerns
> (shared types, integration tests, dev orchestration).

## What this repo is

*(One sentence. Per-repo override required.)*

## How to run it (both tiers)

*(Backend + frontend startup commands; dev orchestration script if any
e.g., docker-compose; how to bring everything up at once.)*

## Project structure

*(Top-level split — typically `src/` or `backend/` + `web/` or `client/`.
Mention which folder is for what so agents don't have to guess.)*

---

## Backend section

*(Mirror sections from `CLAUDE.dotnet-backend.md`. Keep terse — link to
`docs/backend.md` for deeper detail if the template gets too long.)*

## Frontend section

*(Mirror sections from `CLAUDE.web-frontend.md`. Same advice on length.)*

---

## Cross-tier concerns

### Shared types / contracts

*(How backend DTOs are kept in sync with frontend types — generated
OpenAPI client? Manual mirroring? `nswag`? `Refit`?)*

### Integration / E2E testing

*(Tests that exercise both tiers — where they live, how to run them, what
they expect to be running.)*

### Authentication flow

*(How auth is wired end-to-end — token storage, refresh, frontend interceptor
behavior.)*

## Don't

*(Aggregated anti-patterns, both tiers.)*

## Repo-specific overrides

*(Empty in the template.)*
