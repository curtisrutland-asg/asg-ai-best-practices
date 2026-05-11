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
- `[ ]` **Source control** — branching model, commit message format,
  PR etiquette, review expectations.
- `[~]` **Build & run commands** — spine confirmed 2026-05-11. Per-repo
  README is canonical; `dotnet` CLI is the universal interface; local DB is
  SQL Server Developer Edition; Azure-preferred external services; no git
  hooks (CI is the gate); local secrets in gitignored
  `appsettings.Development.json` (migration to `dotnet user-secrets`
  under consideration — see `decisions/0001-local-secrets-storage.md`).
- `[ ]` **Logging & telemetry** — log levels, structured logging fields,
  what to log, what not to log (PII).
- `[ ]` **Error handling** — exception philosophy, when to throw vs return,
  global error handlers.
- `[ ]` **Secrets & configuration management** — local dev, CI, prod;
  user-secrets, key vault, env files.
- `[ ]` **CI/CD expectations** — enough for agents not to break the pipeline
  (full DevOps scope deferred).
- `[ ]` **Documentation expectations** — XML docs, README, ADRs, comments.
- `[ ]` **Dependency management & version pinning** — NuGet, npm, lockfiles,
  upgrade cadence.

## .NET backend-specific

- `[ ]` **Solution / project layout** — e.g., Domain / Application /
  Infrastructure / API split; project naming.
- `[ ]` **Dependency injection** — lifetime conventions, registration
  patterns, anti-patterns to avoid.
- `[ ]` **API design** — REST conventions, versioning, response shape,
  error envelope, status code usage.
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
