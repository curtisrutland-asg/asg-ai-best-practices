# 0007 — Server-rendered web framework: Razor Pages vs MVC + Views

**Status:** Accepted
**Date:** 2026-05-11
**Deciders:** Curtis Rutland (ASG)

## Context

ASP.NET Core offers two paradigms for server-rendered HTML:

- **Razor Pages** — page-centric, with the view (`.cshtml`) colocated
  alongside a `PageModel` class (`.cshtml.cs`). Each page owns its
  own routing and handler methods.
- **MVC + Views** — controller-centric, with controllers in
  `Controllers/`, views in `Views/<Controller>/`, and explicit routing.

The api-design topic answered the *API endpoint* question (MVC
Controllers, not Minimal APIs), but the server-rendered HTML question
was not asked there — they are independent choices in modern ASP.NET
Core.

This gap was surfaced 2026-05-11 when generating an example CLAUDE.md
for a Razor project required making an assumption that wasn't backed
by research.

## Options considered

### Option 1: Razor Pages (chosen)

- **Pros**: Page-centric organization keeps the view and its handler
  code adjacent. Less ceremony than MVC for typical CRUD/form flows.
  Modern Microsoft-recommended pattern for new server-rendered apps.
  Cleaner for small-to-mid scoped server-rendered features.
- **Cons**: Less natural fit for complex multi-step flows that span
  many handlers; routing is less centralized than MVC's attribute
  routing.

### Option 2: MVC + Views

- **Pros**: Familiar to long-time ASP.NET developers. Controllers can
  centralize complex routing logic. Same paradigm as the API
  controllers (single mental model: "everything is a controller").
- **Cons**: More files / more ceremony for simple pages. View location
  conventions (`Views/Controller/Action.cshtml`) split related files
  across folders. Microsoft positions Razor Pages as the modern
  default for server-rendered HTML.

### Option 3: Mix freely

- Both Razor Pages and MVC + Views in the same project, per developer
  preference per feature.
- **Cons**: Inconsistent within a single repo; agents reading a
  partially-Razor-Pages codebase would have to detect which paradigm
  applies per file.

## Decision

**Razor Pages** is ASG's default for server-rendered web apps.

**Mixing with MVC Controllers is acceptable** when controllers handle
the JSON API surface (under `/api/...`) — that pairs naturally with
the api-design topic's "MVC Controllers, not Minimal APIs" decision.
Mixing Razor Pages with MVC *Views* (for server-rendered HTML) in the
same repo is **discouraged** — a repo should pick one paradigm for its
HTML rendering.

## Consequences

- Agents creating new ASG server-rendered apps scaffold with
  `dotnet new webapp` (Razor Pages), not `dotnet new mvc`.
- The `topics/server-rendered-web.md` topic owns Razor Pages-specific
  conventions (page handlers, tag helpers, anti-forgery, layouts,
  view components, model binding). Most of those are still open.
- An ASG Razor Pages app that also exposes JSON APIs:
  - Razor Pages for server-rendered routes
  - MVC Controllers under `/api/...` for JSON endpoints
  - Both follow their respective topic conventions
- Existing MVC + Views apps are not retroactively migrated — they're
  legacy with respect to this decision but are not blocked.
- The example file `examples/CLAUDE.razor-pages.md` is no longer
  making an unverified assumption about framework choice.
