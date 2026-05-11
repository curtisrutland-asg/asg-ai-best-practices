# Topic: Repo scaffolding (starter files)

**Status:** `[~]` drafted — spine confirmed 2026-05-11. README starter
outline still open. Private NuGet feed adoption logged as
`decisions/0006`.

**Scope:** The files every new ASG repo gets by convention — `.gitignore`,
`.gitattributes`, `.editorconfig`, `Directory.Build.props`, `global.json`,
`nuget.config`, `README.md` starter outline, `LICENSE` / `CODEOWNERS` if
applicable. This topic owns the *files themselves*; the **rules** they
encode live in their respective topics (coding-style, source-control,
build-and-run, etc.), and this topic cross-references them.

---

## Interview questions

1. **`.gitignore` baseline** — Microsoft's `dotnet new gitignore` template,
   GitHub's VisualStudio template, or an ASG-custom starter? Plus any
   ASG-specific patterns layered on top
   (`appsettings.Development.json`, `*.user`, `.vs/`, etc.).
2. **Line-ending policy** (`.gitattributes`) — LF everywhere
   (`* text=auto eol=lf`), CRLF for Windows-only repos, or `text=auto`
   (let git decide)? How is this interaction with the build-and-run
   topic's "warnings about CRLF" handled?
3. **`Directory.Build.props`** — does ASG centralize common MSBuild
   settings (TargetFramework, Nullable, LangVersion, ImplicitUsings,
   TreatWarningsAsErrors, etc.) in a single `Directory.Build.props`
   at the solution root, or does each `.csproj` declare its own?
4. **`global.json`** — used to pin the .NET SDK version per repo?
   Pin exact (`10.0.100`) or floating (`rollForward: latestMinor`)?
5. **`.editorconfig`** — Microsoft default + ASG overrides, or
   custom-from-scratch? Where do the conventions from `coding-style.md`
   get encoded (file-scoped namespaces, nullable, brace style, etc.)?
6. **`nuget.config`** — public nuget.org only, or does ASG host an
   internal package feed (Azure Artifacts, GitHub Packages, etc.) that
   needs to be wired into every repo?
7. **`README.md` starter outline** — required section list (Overview,
   Run locally, Build, Test, Architecture, etc.)?
8. **`LICENSE`** — internal-only ASG repos: is a license file required?
   Standard internal-use license header on source files?
9. **`CODEOWNERS`** — used for GitHub PR review routing? Per-repo or
   per-org defaults?

---

## Convention

**Confirmed 2026-05-11:**

- **`.gitignore`** — generate from `dotnet new gitignore` as the baseline,
  then append ASG-specific patterns:
  - `appsettings.Development.json` and any `appsettings.*.Development.json`
    variants — the local-secrets file from the build-and-run topic.
  - OS clutter: `Thumbs.db`, `.DS_Store`.
  - The Microsoft template already covers IDE/build artifacts
    (`.vs/`, `*.user`, `bin/`, `obj/`, JetBrains, etc.) — don't
    re-declare them.
- **`.gitattributes`** — `* text=auto eol=crlf`. ASG is Windows-first;
  force CRLF line endings on all text files at checkout. This silences
  the "LF will be replaced by CRLF" warnings from `git add` on a fresh
  Windows clone.
- **`Directory.Build.props`** — **not used.** ASG explicitly avoids
  centralizing common MSBuild settings. Each `.csproj` declares its
  own `TargetFramework`, `Nullable`, `LangVersion`, `ImplicitUsings`,
  etc. **Implication for agents adding a new project**: copy the
  settings from a sibling `.csproj` rather than relying on inheritance.
- **`global.json`** — used. Floating `rollForward: latestFeature`
  pinning. Locks the major/minor SDK band but accepts newer feature/
  patch SDKs. Bump the floor version deliberately as ASG adopts new
  LTS releases.
- **`.editorconfig`** — Microsoft default (`dotnet new editorconfig`)
  as the base, with ASG overrides layered on top to encode the
  coding-style topic's rules:
  - File-scoped namespaces enforced.
  - C# modern-features preferences (pattern matching, switch
    expressions, target-typed `new`, primary constructors) at
    `suggestion` or `warning` severity.
  - `end_of_line = crlf` to match `.gitattributes`.
  - Naming rules: leave the MS defaults; ASG's naming conventions
    (kebab-case routes, `Async` suffix rules) are project-level rather
    than editor-level and don't get enforced here.
- **`nuget.config`** — **today**: public nuget.org only.
  **Future**: an ASG-hosted private feed (likely Azure Artifacts, matching
  the Azure-preferred stance) is under discussion and will become
  required once ASG starts shipping internal libraries. See
  `decisions/0006-asg-private-nuget-feed.md`. Until the feed lands,
  every repo's `nuget.config` lists only the public source.
- **`LICENSE`** — not used. ASG repos are internal-only and ship
  without a license file.
- **`CODEOWNERS`** — not used. PR review routing is manual today.

**Open / pending follow-up:**

- README.md starter outline — what sections does every ASG repo's
  README include? Not yet decided.

**Cross-references** (the rules these files encode):

- `.editorconfig` enforces conventions from `topics/coding-style.md`.
- `.gitignore` enforces the secrets policy from `topics/build-and-run-commands.md`.
- `.gitattributes` interacts with `topics/source-control.md` (commits,
  line-ending consistency across CI and dev boxes).

## Rationale

- **`dotnet new gitignore` baseline** is well-maintained by Microsoft
  and covers the entire .NET tooling surface (VS, Rider, dotnet CLI,
  NuGet, MSBuild). Reinventing it would duplicate effort and miss
  edge cases (e.g., publish artifacts, ASP.NET Core scaffolded files).
  ASG only needs to add patterns Microsoft's template doesn't cover —
  primarily the local-secrets file.
- **CRLF line endings** reflect ASG's Windows-first reality. Forcing
  LF would introduce friction for Visual Studio devs and produce diff
  noise. The trade-off: non-Windows contributors (rare at ASG today)
  would see CRLF in checkouts. If that changes, this rule revisits.
- **No `Directory.Build.props`** is a deliberate choice for
  transparency over DRY. Centralizing settings in `Directory.Build.props`
  saves a few lines per csproj but hides where settings come from —
  agents (and humans) reading a `.csproj` see "what does this project
  actually compile as?" answered in-file. The cost is duplication
  across projects when ASG bumps `TargetFramework`; ASG accepts that.
- **`global.json` with `latestFeature` roll-forward** strikes the
  balance between reproducible toolchain ("everyone uses .NET 10")
  and patch-uptake ("but if 10.0.200 fixes a bug, it's fine"). Exact
  pinning would force coordinated SDK upgrades; `latestMajor`
  roll-forward defeats the purpose of pinning at all.
- **`.editorconfig` = MS default + ASG overrides** lets agents and
  IDEs enforce the modern-C# rules automatically. Without the
  `.editorconfig`, the coding-style topic's rules become best-effort
  human enforcement. With it, the IDE highlights violations on save.
- **Public-only NuGet today** reflects ASG's current package situation
  — no internal libraries yet shipped. The private-feed ADR captures
  the transition plan so agents know not to invent feed URLs ad hoc.
- **No LICENSE / CODEOWNERS** keeps overhead low for internal-only
  repos. Adding them reflexively (because they're "best practice" in
  open-source contexts) would be cargo-culting; ASG hasn't established
  the need.

## Examples

**`.gitignore` ASG addition block** (append to `dotnet new gitignore`
output):

```gitignore
# === ASG additions ===

# Local secrets (see topics/build-and-run-commands.md)
appsettings.Development.json
appsettings.*.Development.json

# OS clutter
Thumbs.db
.DS_Store
```

**`.gitattributes`:**

```gitattributes
# ASG is Windows-first. Force CRLF on all text files at checkout.
* text=auto eol=crlf
```

**`global.json`:**

```json
{
  "sdk": {
    "version": "10.0.100",
    "rollForward": "latestFeature"
  }
}
```

**`nuget.config`** (current — public only):

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```

**`.editorconfig` (key ASG-override section, after the `dotnet new editorconfig`
baseline):**

```ini
# === ASG C# overrides on top of MS defaults ===

[*.cs]
end_of_line = crlf

# File-scoped namespaces enforced
csharp_style_namespace_declarations = file_scoped:warning

# Modern C# features encouraged
csharp_style_pattern_matching_over_is_with_cast_check = true:suggestion
csharp_style_pattern_matching_over_as_with_null_check = true:suggestion
csharp_style_prefer_switch_expression = true:suggestion
csharp_style_prefer_pattern_matching = true:suggestion
csharp_style_expression_bodied_methods = when_on_single_line:suggestion
csharp_style_expression_bodied_properties = when_on_single_line:suggestion
csharp_style_inlined_variable_declaration = true:suggestion
csharp_style_throw_expression = true:suggestion
csharp_prefer_simple_using_statement = true:suggestion

# Targeted `record` preference for types that have no behavior
# (DTOs / value-shaped types — see coding-style.md)
# Note: no editorconfig rule enforces "use record"; this is a manual
# convention.
```

## Anti-patterns

- **Don't** ship a `.gitignore` that doesn't include
  `appsettings.Development.json`. Secrets must stay out of the repo.
- **Don't** use `eol=lf` in `.gitattributes` for an ASG repo. CRLF
  is the standard.
- **Don't** introduce a `Directory.Build.props` to centralize csproj
  settings. ASG explicitly avoids it; each csproj is self-contained.
- **Don't** pin the SDK with `rollForward: latestMajor` — that's
  effectively no pin.
- **Don't** add a private NuGet feed URL to `nuget.config` before
  the ADR (`decisions/0006`) lands. If you need an internal package
  today, surface the question; don't invent a feed config.
- **Don't** add `LICENSE` or `CODEOWNERS` to a new ASG repo
  reflexively. They're not used today.
- **Don't** put coding-style rules in `.editorconfig` that contradict
  `topics/coding-style.md`. The topic is the policy authority; the
  `.editorconfig` enforces it.
