# Topic: Build & run commands

**Status:** `[~]` drafted — spine confirmed 2026-05-11. Frontend-specific
commands deferred until framework topic is settled. Per-repo command
specifics belong in each repo's CLAUDE.md (no company-wide bootstrap script).

**Scope:** The minimal "how do I start this?" commands every ASG repo's
CLAUDE.md must include. Agents can't do anything useful in a repo until
they can build, run, and test the code.

---

## Interview questions

Backend (.NET):

1. What's the canonical command to restore dependencies — `dotnet restore`
   alone, or wrapped in a script?
2. What's the canonical command to build — `dotnet build`,
   `dotnet build --no-restore`, or via solution file explicitly?
3. How does an agent run the API/service locally — `dotnet run --project ...`?
   Visual Studio / Rider only? A `Makefile` / `taskfile` / PowerShell script?
4. How does an agent run tests — `dotnet test`, with filters, with coverage?
5. Database setup for local dev — LocalDB? Docker? SQL Server Express?
   Migration command(s)?
6. Are there required services (Redis, message bus, etc.) and how are they
   started locally — Docker Compose?
7. Is there a pre-commit or pre-push hook agents must respect?

Frontend (web):

1. Package manager — `npm`, `pnpm`, `yarn`? Locked to a version?
2. Install command — `npm ci`, `npm install`, `pnpm install`?
3. Dev server start — `npm run dev`? `npm start`?
4. Build command — `npm run build`?
5. Test command — `npm test`? `npm run test:unit`? Separate E2E command?
6. Lint and format commands — `npm run lint`, `npm run format`?

General:

1. Is there a single top-level script that does "everything from clean"?
   (`./scripts/bootstrap.ps1`?)
2. What's the absolute minimum required to verify a change before commit?
   (build + test? lint + build + test?)
3. Any commands that are *forbidden* in CI / on shared branches?
   (e.g., `dotnet ef database update` against prod conn string)

---

## Convention

**Confirmed 2026-05-11:**

- **No company-wide bootstrap script.** ASG does not standardize a single
  `bootstrap.ps1` / `make up` / `task dev` command. Each repo's `README.md`
  (and its per-repo `CLAUDE.md`) lists the commands needed to bring it up.
  **Agents should read the repo's README first** before running anything.
- **Canonical .NET CLI**, not IDE-only. Every ASG repo must be runnable
  from the `dotnet` CLI — `dotnet restore`, `dotnet build`,
  `dotnet run --project <path>`, `dotnet test`. Agents do not require
  Visual Studio / Rider.
- **Local database**: SQL Server **Developer Edition**, installed locally
  on each dev box. Not LocalDB, not Docker, not Express. Connection
  string assumes `Server=(local)` or `Server=localhost` with Windows
  integrated auth (`Trusted_Connection=True`).
- **External services** (queues, blobs, cache, etc.) are added on a
  per-project basis. **Azure-managed equivalents are preferred** when a
  service is needed (Azure Service Bus, Azure Storage, Azure Cache for
  Redis, etc.). There is no required baseline list — only add what the
  project actually uses, and reach for Azurite / local emulators for dev.
- **Local-only secrets / config**: `appsettings.Development.json`,
  gitignored at the repo level. Agents add new local-only settings here
  rather than to checked-in `appsettings.json`. **Open decision**: ASG
  is evaluating a migration to `dotnet user-secrets` — see
  `decisions/0001-local-secrets-storage.md`.
- **Git hooks**: none. ASG does not install pre-commit / pre-push hooks
  via Husky or similar. **CI is the only gate.** Agents should not assume
  a local check will catch a broken build before push.
- **CI gates** (the "definition of done" for a change):
  - `dotnet build` must succeed (zero errors).
  - All tests must pass. (No tests exist as a baseline yet — when the
    testing topic is settled, this becomes enforceable. Today: don't
    introduce failing tests; if no tests exist yet, the build gate is
    the bar.)
  - No format-check, lint, or coverage gate today.

## Rationale

- **Per-repo README over standard bootstrap script** reflects ASG's
  reality: project shapes vary too much for one script to fit. The cost
  is that agents must read before acting — saying "here's the universal
  command" would be wrong.
- **`dotnet` CLI as the canonical interface** means agents (and CI) get
  the same path. IDE-only workflows are fine for humans but can't be
  scripted.
- **Local SQL Server Developer Edition** gives the closest possible
  parity to prod SQL Server without licensing fees. LocalDB / Express
  have feature gaps; Docker adds per-dev friction. The trade-off is a
  one-time install on each dev box.
- **Azure-preferred external services** aligns with ASG's deployment
  target — using the same service family locally (via emulators) and in
  prod reduces config drift.
- **CI-only enforcement** keeps local development fast and avoids hook
  failures blocking commits mid-thought. The trade-off is that PRs can
  fail CI for things a hook would have caught — agents compensate by
  running `dotnet build` and `dotnet test` themselves before pushing.

## Examples

**Minimum-viable command set for an ASG backend repo:**

```pwsh
# 1. Restore + build
dotnet restore
dotnet build

# 2. Run the API (path depends on the repo's layout — check the sln)
dotnet run --project src/ASG.Billing.Api

# 3. Run tests (testing conventions TBD; works once tests exist)
dotnet test

# 4. Apply EF Core migrations (data-access topic will refine this)
dotnet ef database update --project src/ASG.Billing.Infrastructure --startup-project src/ASG.Billing.Api
```

**Setting a local secret:**

Edit `src/ASG.Billing.Api/appsettings.Development.json` (gitignored). Do
**not** put secrets in `appsettings.json` (checked in).

```jsonc
// appsettings.Development.json — gitignored
{
  "ConnectionStrings": {
    "Default": "Server=(local);Database=ASG.Billing.Dev;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "AzureServiceBus": {
    "ConnectionString": "<your local-dev SB connection string or emulator>"
  }
}
```

## Anti-patterns

- **Don't** require an IDE to build or run a repo. If `dotnet build` from
  the CLI fails, that's a bug.
- **Don't** introduce a custom bootstrap script unless the repo's README
  documents and justifies it.
- **Don't** check secrets into `appsettings.json`. Use
  `appsettings.Development.json` (gitignored) for local-only values.
- **Don't** add a `docker-compose.yml` "for everything" by default. Add
  containers only for services the project actually uses, and prefer
  Azure-managed services (with Azurite-style emulators locally).
- **Don't** rely on a pre-commit hook to catch bad changes — ASG doesn't
  install them. Run `dotnet build` and `dotnet test` yourself before
  pushing.
- **Don't** push a change that breaks `dotnet build` or introduces a
  failing test. These are CI gates and will block the PR.
