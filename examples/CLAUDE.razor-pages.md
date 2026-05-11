# CLAUDE.md — ASG Razor Pages web app

> **Example, not a maintained template.** Generated 2026-05-11 by
> synthesizing the current `topics/*.md` content in the
> `ai-best-practices` research repo. The Razor Pages framework choice
> is backed by `decisions/0007`. Razor-specific *conventions* (Page
> handler patterns, tag helpers, anti-forgery, layouts, view
> components, model binding) are **not yet researched** and are not
> encoded here — they are tracked in `topics/server-rendered-web.md`.

## What this repo is

Internal ASG Razor Pages web application on .NET 10. MPA — frontend
lives inside the web project (`Pages/`, `wwwroot/`). Single deployable
artifact.

## How to run it

```pwsh
dotnet restore
dotnet build
dotnet run --project src/ASG.<Product>.Web
```

- Local DB: SQL Server **Developer Edition**, installed locally
  (`Server=(local);Trusted_Connection=True;TrustServerCertificate=True`).
- Local secrets: edit `appsettings.Development.json` (gitignored).
  Never commit it.
- No bootstrap script; commands above are the canonical entry points.
- Visual Studio / Rider optional — `dotnet` CLI is the contract.

## Project structure

- `.sln` at repo root.
- Web project: `ASG.<Product>.Web`.
- Razor Pages: `Pages/<Feature>/` (PageModel `.cshtml.cs` next to view
  `.cshtml`).
- Shared layouts / partials: `Pages/Shared/`.
- Static assets: `wwwroot/`.
- Tests: location TBD (testing topic is unsettled at ASG).
- **No `Directory.Build.props`** — every `.csproj` declares its own
  `TargetFramework`, `Nullable`, etc.
- **Mirror the existing layout.** Don't refactor solution structure
  without asking.

## Naming

- **Namespaces** match csproj + folder structure. File
  `Pages/Orders/Index.cshtml.cs` → `namespace ASG.<Product>.Web.Pages.Orders;`.
- **File-scoped namespaces** always (`namespace Foo.Bar;`), never
  block-scoped.
- **Interfaces** always `I`-prefixed: `IOrderService`, never `OrderService`
  for the abstraction.
- **`Async` suffix** only when a sync counterpart exists. Diverges from
  the BCL "always suffix" rule — apply ASG's version inside ASG repos.
- **DTOs at API boundaries** (if any APIs exist): `<Operation>Request` /
  `<Operation>Response`. Internal layer-crossing types: `<Concept>Dto`.
  Razor PageModels are PageModels, not DTOs — properties live on the
  `PageModel` class.
- **One public type per file**; filename matches the type name. Small,
  closely-related types (records, enums) may share a file.
- **URL/route casing** (any API or non-Razor route): `kebab-case`,
  lowercase. Razor Page routes follow the folder structure.

## Coding style

- Target **.NET 10 LTS**. Use modern C# features by default: `record`,
  switch expressions, pattern matching, primary constructors,
  target-typed `new`, collection expressions, expression-bodied members
  where they read clearly.
- `<Nullable>enable</Nullable>` in every `.csproj`.
- `TreatWarningsAsErrors` **disabled**, but don't let warnings accumulate.
- **`record` by default** for DTOs / value-shaped types;
  use `class` only when reference identity, inheritance, or mutability
  is genuinely needed (EF entities, services with state).
- Reduce indentation: guard-clause early returns over nested `if`;
  switch expressions over `if/else if` chains.
- Microsoft's C# coding conventions are the base. ASG's `.editorconfig`
  (MS default + ASG overrides) is the enforcement layer.

## Errors

- **Throw, don't return.** No `Result<T>` / `OneOf<T, Error>` baseline.
  Expected-but-erroneous conditions (not-found, validation,
  business-rule violations) are exceptions.
- **BCL exception types first**: `KeyNotFoundException`,
  `ArgumentException`, `InvalidOperationException`,
  `UnauthorizedAccessException`. Add a custom domain type
  (`OrderNotFoundException`, `InsufficientFundsException`) only when the
  name carries domain meaning.
- **Global handler**: `IExceptionHandler` (.NET 8+ pattern).
  Register with `services.AddExceptionHandler<TYourHandler>()` and
  `app.UseExceptionHandler()`.
- **Re-throw**: `throw;` to preserve stack, or wrap
  (`throw new XException("context", ex)`) to add domain context.
  **Never `throw ex;`** — it clobbers the stack.
- **API error envelope** (for any JSON endpoints): `ProblemDetails`
  (`application/problem+json`) is the working default until a final
  decision lands.

## Logging

- Inject `ILogger<T>`. ASG uses the **built-in** logger by default; no
  Serilog or OpenTelemetry installed unless a client requires it.
- **Always use message templates with named placeholders**:

  ```csharp
  logger.LogWarning("Pricing fell back to cache for {ProductId}", productId);
  ```

  Never `logger.LogWarning($"... {productId}")` — interpolated strings
  destroy structured fields.
- **No PII or secrets in logs, ever.** Log IDs (`{OrderId}`,
  `{CustomerId}`); never names, emails, addresses, tokens, full bodies.
- **Catch sites don't log.** The global `IExceptionHandler` is the
  single logger for exceptions. Only log at a catch site if you're
  enriching with context the handler can't see — usually rare.
- **Production minimum level: Warning.** Information-level logs are for
  local dev; assume they won't appear in prod.
- **Telemetry**: Azure Application Insights direct SDK
  (`Microsoft.ApplicationInsights.AspNetCore`). No
  `UseHttpLogging` / custom request middleware — App Insights
  auto-instruments requests.

## API endpoints (if any)

Razor Pages handles server-rendered routes. If this repo also exposes
JSON APIs (e.g., for AJAX from the Razor pages):

- **MVC Controllers**, not Minimal APIs. `[ApiController]` + attribute
  routing under `/api/...`.
- **camelCase JSON** (`orderId`, not `OrderId` or `order_id`).
- **Versioning**: `X-Api-Version` header. URL stays versionless
  (`/api/orders`, not `/api/v2/orders`). Wire via `Asp.Versioning.Mvc`.
- **HTTP methods**: GET, POST, PUT, DELETE only. **No PATCH** — PUT
  handles both full replacement and partial update.
- **Success codes**: 200 OK for read/update/create (with body), 204 for
  DELETE, 202 for async-accepted. **Not 201 Created** for POST.
- **List responses** (working default): paginated envelope
  `{ items, totalCount, page, pageSize, hasNextPage }`.

## Source control

- **GitHub Flow**: short-lived branches off `main` → PR → **squash-merge**.
- **Branches**: lowercase, type-prefixed:
  `feature/<slug>`, `fix/<slug>`, `chore/<slug>`, etc. Casing is
  enforced consistent (Visual Studio misbehaves on mixed-case branches).
- **Commits and PR titles**: Conventional Commits — `<type>(<optional-scope>): <subject>`.
  Lowercase imperative subject, no trailing period, ~72 chars.
- **PR title becomes the squash commit subject** verbatim — write it
  in CC format.
- **Local enforcement**: none. CI is the only gate. Run
  `dotnet build` (and tests, when present) before pushing.

## Testing

**No ASG-wide testing decisions exist yet.** This repo's tests follow
whatever conventions the repo's owner set. CI gates on `dotnet build`
succeeding and (when tests exist) all tests passing. Don't introduce
failing tests.

## Repo scaffolding (already in place; agents don't usually edit)

- `.gitignore`: `dotnet new gitignore` baseline + `appsettings.Development.json`,
  `Thumbs.db`, `.DS_Store`.
- `.gitattributes`: `* text=auto eol=crlf`.
- `.editorconfig`: MS default + ASG overrides.
- `global.json`: SDK pinned with `rollForward: latestFeature`.
- `nuget.config`: public nuget.org only.
- No `Directory.Build.props`, `LICENSE`, or `CODEOWNERS`.

## Don't

- Don't introduce Minimal APIs; use MVC Controllers if you need APIs.
- Don't use `PATCH`; PUT handles full and partial updates.
- Don't return `201 Created` from POST endpoints; return `200 OK` with
  the body.
- Don't add the `Async` suffix to every async method — only when a sync
  counterpart exists.
- Don't use block-scoped namespaces; file-scoped only.
- Don't refactor the solution structure without asking. Mirror what's there.
- Don't introduce `Directory.Build.props` to centralize csproj settings.
- Don't introduce Serilog, OpenTelemetry, or `Result<T>` — the ASG
  defaults are built-in `ILogger<T>`, App Insights direct SDK, and
  throw-liberal exceptions.
- Don't log PII or secrets. Ever.
- Don't use interpolated strings in `Log*` calls — loses structured fields.
- Don't `throw ex;` — destroys the stack. Use `throw;` or wrap.
- Don't put `appsettings.Development.json` in the repo. Use the
  gitignored local copy for local-only values.
- Don't add `v1`/`v2` to API URLs; version via the `X-Api-Version` header.
- Don't write commit messages in past tense (`added`, `fixed`) — use
  imperative (`add`, `fix`).

## Repo-specific overrides

*(Empty in this example. Each real repo fills this in with what's
unique: domain conventions, third-party SDKs, deployment specifics,
auth scheme particulars, Razor-specific patterns the repo adopts.)*

---

## Gaps surfaced by generating this example

The following are **not yet covered in the ai-best-practices research** —
this template makes no claims on them:

- **Razor-specific conventions**: Page handler naming
  (`OnGet`/`OnPost`/`OnGetAsync`), tag helpers vs HTML helpers,
  anti-forgery token policy, layout/partial structure, view component
  use, model binding edge cases. **Tracked**: `topics/server-rendered-web.md`.
  Framework choice (Razor Pages) is itself confirmed —
  `decisions/0007-razor-pages-vs-mvc-views.md`.
- **Data access**: EF Core conventions, migration workflow, repository
  pattern preferences (data-access topic not yet researched).
- **AuthN/AuthZ**: cookie auth, identity providers, policies (auth topic
  not yet researched).
- **Validation**: FluentValidation vs DataAnnotations (validation topic
  not yet researched).
- **Testing**: every aspect (topic explicitly unsettled).
- **API error envelope**: still an open ADR (decisions/0002).
- **Exception → status code mapping**: still an open ADR (decisions/0003).
- **List response shape**: still an open ADR (decisions/0005).
