# Topic: Project structure & repo layout

**Status:** `[~]` drafted — spine confirmed 2026-05-11; secondary items
(top-level folders, README/gitignore starter, untouchable folders, SPA
frontend convention) still open. Promote to `[x]` once those are decided.

**Scope:** How an ASG repo is organized on disk. Top-level folders, where
source / tests / docs / scripts live, monorepo vs polyrepo defaults.

---

## Interview questions

Backend (.NET):

1. Do ASG backend repos follow a layered solution structure (Domain /
   Application / Infrastructure / API)? Or vertical-slice / feature-folder?
   Or "it depends, here's the rule of thumb"?
2. Where does the `.sln` file live? At repo root, or inside a `src/` or
   `Source/` folder? (The OMSNext repo uses `Source/`, but that's not ASG.)
3. Project (`.csproj`) naming convention — e.g., `ASG.<Product>.<Layer>`?
4. Where do tests live — sibling `*.Tests` projects, a separate `tests/`
   folder, or both?
5. Is there a standard top-level `scripts/` folder for dev tooling? `tools/`?
6. Where does infrastructure-as-code live (if any)? `infra/`? `deploy/`?
7. Where do docs live? Top-level `docs/`? In-project? On Confluence/SharePoint
   (i.e., not in the repo at all)?

Frontend (web):

1. Is the frontend in the same repo as the backend, or separate?
2. If same repo: where does it live? `web/`, `client/`, `frontend/`, `src/Web/`?
3. Standard component organization — by feature, by type (components/pages/hooks),
   or framework-default?
4. Where do shared utilities / hooks / types live?
5. Public assets — `public/`, `wwwroot/`, other?

General:

1. Monorepo or polyrepo by default? When does each apply?
2. Are there any folders/files an agent should *never* touch
   (generated code, vendored dependencies, etc.)?
3. Is there a standard `.gitignore` / `.gitattributes` ASG repos start from?
4. What about a standard `README.md` outline at the repo root?

---

## Convention

**Confirmed (2026-05-11):**

- **`.sln` location**: at repo root. Not inside `src/` or `Source/`.
- **Default repo strategy**: monorepo — backend and web frontend live in
  the same repo for full-stack apps.
- **Backend solution layout**: no firm ASG-wide rule. The tech lead picks
  per project. **Agents must mirror whatever layout already exists in the
  repo** rather than introducing a new structure. Do not refactor the
  solution structure without asking.
- **Frontend location in a monorepo**:
  - **MPA (Razor Pages / MVC / server-rendered)**: frontend assets live
    *inside* the API project — `wwwroot/` for static assets, `Views/` for
    Razor templates. This is the default for ASG MPA apps.
  - **SPA (React / Angular / Blazor WASM / etc.)**: layout decision is
    **open** — colocated inside the API (`ClientApp/`) and separate
    sibling folder (`web/`, `client/`) are both viable. Per-repo
    decision until ASG-wide convention is set. See open decision below.
- **`.csproj` naming pattern**: `ASG.<Product>.<Layer>`. Examples:
  `ASG.Billing.Api`, `ASG.Billing.Domain`, `ASG.Billing.Infrastructure`,
  `ASG.Billing.Tests`.
- **Test project location**: **deferred** — testing is its own topic with
  open decisions. See `topics/testing.md`.

**Open (will revisit):**

- SPA frontend folder convention — log to `decisions/` once the frontend
  framework choice is settled.
- Top-level `scripts/`, `tools/`, `infra/`, `docs/` folder conventions —
  not yet established.
- Files/folders agents should never touch (generated code, vendored deps) —
  not yet established. Default: leave anything obviously generated alone
  and confirm before touching.

**Moved to other topics:**

- `.gitignore` / `.gitattributes` / `.editorconfig` / `global.json` /
  `nuget.config` and other repo-root scaffolding files — see
  `topics/repo-scaffolding.md`.
- `README.md` starter outline — currently open in
  `topics/repo-scaffolding.md`.

## Rationale

- **`.sln` at root** keeps the `dotnet` CLI entry point unambiguous —
  `dotnet build` from repo root just works, no `-p`/`-s` needed.
- **No firm backend layout rule** reflects that ASG's projects span a wide
  size range; forcing one structure would waste effort on small CRUD apps
  and constrain complex ones. The cost is that agents must observe before
  acting — "look at the existing project structure before adding a new
  project."
- **Frontend inside the API project for MPAs** matches ASP.NET's default
  templates and keeps a single deployable artifact. SPAs break that
  assumption because the frontend usually wants its own build pipeline.
- **`ASG.<Product>.<Layer>` project names** make assembly origin obvious
  in stack traces and DLL lists, and prevent naming collisions across
  repos.

## Examples

**MPA monorepo (Razor Pages / MVC) — typical layout:**

```
asg-billing/
  ASG.Billing.sln
  ASG.Billing.Api/              ← API project; hosts the MPA frontend too
    wwwroot/                    ← static assets (css, js, images)
    Views/                      ← Razor views (if MVC)
    Pages/                      ← Razor Pages (if Razor Pages)
    Controllers/                ← if MVC
    Program.cs
    ASG.Billing.Api.csproj
  ASG.Billing.Domain/           ← only if the lead chose a layered layout
    ASG.Billing.Domain.csproj
  ASG.Billing.Infrastructure/   ← only if layered
    ASG.Billing.Infrastructure.csproj
  ASG.Billing.Tests/            ← location TBD per testing topic
    ASG.Billing.Tests.csproj
  README.md
  .gitignore
```

**SPA monorepo — layout decision is open per repo.** Two viable shapes
(pick one per repo, mirror it once chosen):

```
# Option A: colocated inside API
asg-billing/
  ASG.Billing.sln
  ASG.Billing.Api/
    ClientApp/                  ← SPA source lives here
    ...

# Option B: sibling folder
asg-billing/
  ASG.Billing.sln
  ASG.Billing.Api/
  web/                          ← or client/ — sibling SPA folder
    package.json
    src/
```

## Anti-patterns

- **Don't** put the `.sln` under a `src/` or `Source/` folder. ASG keeps
  it at repo root.
- **Don't** refactor an existing repo's solution structure (flat ↔ layered
  ↔ vertical slices) without explicit approval. Agents mirror; humans
  decide.
- **Don't** invent a new `.csproj` naming pattern. If you're adding a
  project, follow `ASG.<Product>.<Layer>` and match the existing siblings.
- **Don't** assume the SPA folder convention from one repo applies to
  another — it's per-repo until standardized.
