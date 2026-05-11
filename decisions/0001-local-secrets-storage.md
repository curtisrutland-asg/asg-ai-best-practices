# 0001 — Local secrets storage convention

**Status:** Open — using `appsettings.Development.json` today; `dotnet user-secrets` under consideration.
**Date:** 2026-05-11
**Deciders:** Curtis Rutland (ASG)

## Context

ASG backend projects need a place to store local-only development secrets
(connection strings to local SQL, Azure dev tenant keys, etc.) that don't
get checked into git. Two viable patterns exist in the .NET ecosystem; ASG
currently uses one but wants to evaluate the other.

## Options considered

### Option 1: `dotnet user-secrets` (per-project store outside the repo)

- **Pros**: Lives in `%APPDATA%\Microsoft\UserSecrets\<GUID>\secrets.json`,
  so it can't be accidentally committed. ASP.NET integrates natively in
  Development environment. Industry-standard pattern.
- **Cons**: Slightly less discoverable for new devs (the file isn't in
  the project). Requires `dotnet user-secrets init` per project.

### Option 2 (current): `appsettings.Development.json`, gitignored

- **Pros**: File sits next to the rest of the project config; new devs
  see it immediately. No tooling step to remember.
- **Cons**: One missing `.gitignore` entry and a secret leaks into the
  repo. Easy to accidentally `git add -f`. The file's presence in the
  project tree invites well-meaning "let me add this to source control."

## Decision

**Current**: Option 2 (`appsettings.Development.json`, gitignored) is the
ASG-wide convention as of 2026-05-11.

**Pending evaluation**: ASG is considering moving to Option 1
(`dotnet user-secrets`). This decision entry will be updated when that
evaluation concludes.

## Consequences

Until this is resolved:

- Agents should default to `appsettings.Development.json` for new local
  secrets in existing ASG repos.
- New repos created today should use Option 2 (current convention).
- When the decision lands on Option 1, this entry gets updated and a
  migration plan recorded — existing repos will not be retroactively
  converted unless individually scheduled.
- Either way, agents must verify the secrets file is in `.gitignore`
  before committing changes anywhere near it.
