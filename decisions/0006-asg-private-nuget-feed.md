# 0006 — ASG-private NuGet feed adoption

**Status:** Open — under discussion
**Date:** 2026-05-11
**Deciders:** Curtis Rutland (ASG)

## Context

ASG's current `nuget.config` lists only the public nuget.org feed.
This is sufficient today because ASG doesn't yet publish internal
NuGet packages — every dependency is a public OSS or Microsoft
package.

As ASG begins shipping internal libraries (shared utilities,
ASG-specific abstractions, internal client SDKs, etc.), every consuming
repo will need to resolve those packages from somewhere. The question
is **where**.

This decision is **under discussion** but not yet made — it should land
before the first ASG-published library, otherwise repos will start
inventing ad-hoc solutions (referencing local builds, copy-pasting
source, etc.).

## Options considered

### Option 1: Azure Artifacts feed

Host packages on an ASG-owned Azure Artifacts feed in Azure DevOps
or Azure subscription. Public feed remains in `nuget.config`; the
private feed is added as a second source.

- **Pros**: Matches ASG's Azure-preferred services stance. Tightly
  integrated with Azure AD auth, Azure Pipelines, and ASG's existing
  Azure subscription. Supports upstream-source proxying (private feed
  can proxy nuget.org so consumers only need one feed URL).
- **Cons**: Costs (Azure Artifacts pricing per feed/storage).
  Authentication setup per dev box (PAT or
  `Azure.Identity.AzureCliCredential`).

### Option 2: GitHub Packages

Host packages on GitHub Packages alongside ASG's source repos.
GitHub-issued PAT used for auth.

- **Pros**: Co-located with source. PAT-based auth.
- **Cons**: Diverges from Azure-preferred stance. Less mature than
  Azure Artifacts for .NET-specific workflows.

### Option 3: MyGet / ProGet / other third-party hosted feed

External SaaS or self-hosted feed.

- **Pros**: Mature .NET-specific tooling (ProGet is .NET-native).
- **Cons**: Additional vendor; auth and access control to manage.

### Option 4: No private feed — keep packages as project references

Don't ship NuGet packages internally; share code via project
references in solution files or git submodules.

- **Pros**: No infrastructure. Simple.
- **Cons**: Doesn't scale — shared code becomes copy-paste or
  submodule complexity. Defeats the point of having shared libraries.

## Decision

**Open** as of 2026-05-11. Discussion ongoing.

**Working assumption until decided**: Option 1 (Azure Artifacts) is
the most likely landing spot given ASG's Azure-preferred stance.
Agents should not invent feed URLs or ad-hoc package-sharing schemes
before this decision lands.

## Consequences

- Until decided, every ASG repo's `nuget.config` lists only
  nuget.org. The repo-scaffolding topic's example reflects this.
- When the decision lands, the `repo-scaffolding.md` example
  `nuget.config` gets updated, and existing repos will need a
  one-line addition.
- Authentication mechanism (PAT vs `Azure.Identity` vs Azure CLI)
  is part of the decision and affects local-dev setup
  instructions in build-and-run.
- Existing internal code-sharing solutions (project references,
  git submodules, copy-paste) should be marked for migration once
  the feed is live.
