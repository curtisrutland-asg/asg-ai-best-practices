# Topic Taxonomy

Master checklist of topics for the ASG ai-best-practices guide. Each topic
has a status flag and a one-line scope statement. When picking what to work
on next, prioritize `[ ]` topics highest on the list — they're ordered by
"how often an agent hits this in the first 5 minutes of working in a repo."

Status legend:
- `[ ]` not-started
- `[~]` drafted (file exists in `topics/`, not yet user-confirmed)
- `[x]` confirmed (user has reviewed and approved)
- `[-]` not applicable / deferred

---

## Cross-cutting (apply to both backend and frontend)

- `[~]` **Project structure & repo layout** — spine confirmed 2026-05-11
  (`.sln` at root, monorepo, mirror-existing layout, `ASG.<Product>.<Layer>`
  csproj names, MPA frontend inside API project). Secondary items still open.
- `[~]` **Naming conventions** — backend type/file/namespace/route conventions
  confirmed 2026-05-11. Frontend deferred (framework pending); source-control
  naming carved off to its own topic.
- `[~]` **Testing strategy** — topic file exists but **no decisions made yet**;
  agents reading any template should treat testing as unstandardized
  until this topic is filled in.
- `[~]` **Coding style & formatting** — confirmed 2026-05-11: modern C#
  preferred; .NET 10 LTS for new scaffolds; file-scoped namespaces; reduce
  nesting (guard clauses, switch expressions); MS style guide as base;
  nullable refs enabled; `TreatWarningsAsErrors` disabled; `record` by
  default for DTOs. Open: `.editorconfig` starter, analyzer pack.
- `[~]` **Source control** — spine confirmed 2026-05-11: GitHub Flow,
  lowercase type-prefixed branches (`feature/`, `fix/`, ...), Conventional
  Commits (standard set, scope optional), squash-merge to `main`, PR title =
  future squash subject. Open: force-push rules, PR description template,
  review expectations, stale-branch policy, commit signing.
- `[~]` **Build & run commands** — spine confirmed 2026-05-11. Per-repo
  README is canonical; `dotnet` CLI is the universal interface; local DB is
  SQL Server Developer Edition; Azure-preferred external services; no git
  hooks (CI is the gate); local secrets in gitignored
  `appsettings.Development.json` (migration to `dotnet user-secrets`
  under consideration — see `decisions/0001-local-secrets-storage.md`).
- `[~]` **Logging & telemetry** — confirmed 2026-05-11: built-in
  `ILogger<T>` by default (Serilog escape hatch for client-driven file
  logging needs); Application Insights direct SDK; no PII or secrets
  in logs ever; catch sites don't log (global `IExceptionHandler`
  handles it); always message templates with named placeholders;
  production min level = Warning; no request/response middleware (App
  Insights auto-instruments). Open: correlation ID strategy, custom
  metrics, retention/sampling, local-dev sinks.
- `[~]` **Error handling** — confirmed 2026-05-11: throw-liberally
  (traditional .NET); BCL types + a small set of custom domain exceptions;
  global handler is `IExceptionHandler` (.NET 8+); re-throw with `throw;`
  to preserve stack or wrap to add context (never `throw ex;`). Open:
  error envelope shape (`decisions/0002`); exception→status code mapping
  (`decisions/0003`).
- `[ ]` **Secrets & configuration management** — local dev, CI, prod;
  user-secrets, key vault, env files.
- `[ ]` **CI/CD expectations** — enough for agents not to break the pipeline
  (full DevOps scope deferred).
- `[ ]` **Documentation expectations** — XML docs, README, ADRs, comments.
- `[ ]` **Dependency management & version pinning** — NuGet, npm, lockfiles,
  upgrade cadence.
- `[~]` **Repo scaffolding (starter files)** — spine confirmed
  2026-05-11: `.gitignore` from `dotnet new gitignore` + ASG patterns
  (secrets file, OS clutter); `.gitattributes` forces CRLF
  (Windows-first); no `Directory.Build.props` (each csproj is
  self-contained); `global.json` pins SDK with `latestFeature`
  roll-forward; `.editorconfig` = MS default + ASG overrides encoding
  coding-style rules; `nuget.config` is public-only today, private
  ASG feed under discussion (`decisions/0006`); no LICENSE or
  CODEOWNERS. Open: README starter outline.

## .NET backend-specific

- `[ ]` **Solution / project layout** — e.g., Domain / Application /
  Infrastructure / API split; project naming.
- `[ ]` **Dependency injection** — lifetime conventions, registration
  patterns, anti-patterns to avoid.
- `[~]` **API design** — spine confirmed 2026-05-11: MVC Controllers
  (not Minimal APIs); camelCase JSON; `X-Api-Version` header (URL stays
  versionless); GET/POST/PUT/DELETE only (no PATCH; PUT does both full
  and partial); 200 OK for create with body (not 201), 204 for DELETE.
  Open: list response shape (`decisions/0005`, flagged important),
  OpenAPI, idempotency, ETag, date/time wire format, query syntax.
- `[ ]` **Data access** — EF Core conventions, migrations workflow,
  repository pattern vs direct DbContext.
- `[ ]` **AuthN / AuthZ** — JWT, role/claim conventions, middleware
  ordering, policy registration.
- `[ ]` **Async / await patterns** — `ConfigureAwait`, sync-over-async
  bans, `Task.Run` rules.
- `[ ]` **Cancellation token propagation** — when required, how plumbed.
- `[ ]` **Validation** — FluentValidation vs DataAnnotations; where
  validation lives.
- `[ ]` **Background work** — hosted services, queues, schedulers.
- `[ ]` **Health checks** — readiness/liveness probes, registration.

## Web frontend-specific

- `[ ]` **Framework choice & version** — *open question; need user input.*
- `[ ]` **Folder structure & component organization**
- `[ ]` **State management approach**
- `[ ]` **API client layer** — typed clients, error handling, retries.
- `[ ]` **Routing conventions**
- `[ ]` **Form handling & validation**
- `[ ]` **Styling approach** — CSS modules / Tailwind / component library.
- `[ ]` **Accessibility baseline** — WCAG level expected.
- `[ ]` **Build tooling** — Vite / Webpack / etc.

---

## Recommended first three (Phase B kickoff)

These three are highest-leverage — agents reference them within seconds of
opening a repo:

1. Project structure & repo layout
2. Naming conventions
3. Build & run commands

(Testing was originally on the recommended-three list, but build/run commands
beat it for "first 5 minutes" impact — agents can't do anything until they
can run the code.)
