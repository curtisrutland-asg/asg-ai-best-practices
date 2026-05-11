# Topic: Source control (branching, commits, PRs)

**Status:** `[~]` drafted — spine confirmed 2026-05-11. Force-push rules,
PR description template, and review expectations remain open.

**Scope:** Git workflow conventions ASG expects in every repo —
branching model, branch naming, commit message format, merge strategy,
PR title and merge behavior.

---

## Interview questions

Already captured (see Convention):

- Branching model
- Branch naming convention + casing rule
- Commit message format
- Merge strategy
- PR title format

Still open:

1. Force-push rules — allowed on personal branches? Banned on `main`?
2. PR description template — required sections?
3. Review expectations — required approvals, who reviews, when to merge?
4. Stale-branch policy — auto-delete after merge? Manual cleanup?
5. Signed commits / GPG signing required?

---

## Convention

**Confirmed 2026-05-11:**

- **Branching model: GitHub Flow.** `main` is always deployable. Short-lived
  branches off `main`; one PR per branch; merge back to `main`. No
  long-lived `develop` or `release/*` branches.
- **Branch naming**: type-prefixed, lowercase, slash-delimited.
  - Format: `<type>/<short-slug>`
  - Types match Conventional Commits types: `feature/`, `fix/`,
    `chore/`, `docs/`, `refactor/`, `test/`, `build/`, `ci/`, `perf/`,
    `style/`, `revert/`.
  - Slug is lowercase kebab-case describing the change.
  - **Casing is enforced consistent.** Always lowercase prefix
    (`feature/foo`, not `Feature/foo`). Mixing cases within a repo
    breaks Visual Studio's branch handling — avoid.
- **Commit message format: Conventional Commits**, standard type set,
  **scope optional**.
  - Format: `<type>(<optional-scope>): <subject>`
  - Subject: lowercase verb in imperative mood ("add", "fix", "remove"),
    no trailing period, ~72 chars max.
  - Body (optional but encouraged for non-trivial changes): blank line
    after subject, then free-form prose. Wrap ~72 chars.
  - Standard type set (same as branch prefixes): `feat`, `fix`, `chore`,
    `docs`, `refactor`, `test`, `build`, `ci`, `perf`, `style`, `revert`.
  - Scope (the `(...)` segment) is optional. Use it when the affected
    module/area is non-obvious and worth surfacing.
- **Merge strategy: squash merge** to `main`. Each PR collapses into one
  commit on `main`. Per-commit history on the feature branch is discarded
  at merge time (so agents don't need to keep the branch history clean —
  the squash subject is what survives).
- **PR title = the future squash commit subject.** Author writes the PR
  title in Conventional Commits format; GitHub uses the PR title
  verbatim as the squash commit subject when the PR is merged. This
  means PR titles must follow the commit format rules above (lowercase
  verb, ~72 chars, no period).

**Open / pending follow-up:** see Interview questions section.

## Rationale

- **GitHub Flow over Git Flow** because ASG's deployment cadence and team
  size don't justify long-lived `develop`/`release` branches. `main` is
  the source of truth; everything else is short-lived working state.
- **Lowercase branch prefixes** match Conventional Commits type casing
  (so the branch name and the commit `<type>` agree visually). Also
  matches industry default and avoids the mixed-case issues some Visual
  Studio workflows surface — VS can fail to recognize branches when
  casing varies within a repo's history.
- **Conventional Commits** gives every commit a parseable structure,
  which (a) makes change intent legible at a glance and (b) opens the
  door to auto-changelog / semver tooling later if ASG wants it.
- **Scope optional** because forcing a scope on every commit adds
  friction without much payoff for a small org. Agents should add scope
  when the affected module is non-obvious; skip it when it's clear
  from the subject.
- **Squash merge** keeps `main`'s history linear and one-commit-per-PR.
  This makes `git log main` readable as a feature history, and means
  agents don't need to keep their branch commits cosmetically clean.
- **PR title = squash subject** is a direct consequence of squash-merge
  + Conventional Commits: the PR title is the artifact that survives,
  so it must follow the format.

## Examples

**Branch names:**

```
feature/add-order-export
fix/login-timeout-handling
chore/bump-net-runtime-version
docs/clarify-secrets-storage
refactor/extract-order-pricing
```

**Commit messages:**

```
feat(orders): add CSV export endpoint

Adds GET /api/orders/export which returns the caller's orders as a
CSV stream. Auth scope: customer. No pagination — assumes export is
bounded by date filter.
```

```
fix: handle null user in login flow
```

```
chore: bump .NET runtime to 10.0.1
```

**PR title (becomes the squash commit subject):**

```
feat(orders): add CSV export endpoint
```

Not:

```
Add order export ❌  (free-form, no CC type)
feat: Add CSV export endpoint ❌  (subject is capitalized)
feat(orders): added CSV export endpoint. ❌  (past tense + trailing period)
```

## Anti-patterns

- **Don't** mix cases for branch prefixes within a repo
  (`Feature/foo` and `feature/bar` side by side). Visual Studio
  misbehaves with mixed-case branches; pick lowercase and stick with it.
- **Don't** use free-form commit subjects. Conventional Commits format
  is required.
- **Don't** capitalize the commit subject or end it with a period.
  (`feat: add export` ✓, `feat: Add export.` ✗.)
- **Don't** write commit subjects in past tense ("added", "fixed").
  Imperative mood ("add", "fix") — describes what the commit does, not
  what was done.
- **Don't** create long-lived branches (>~a few days). If a feature is
  too big to land in a few days, split it into smaller PRs behind a
  feature flag or stub.
- **Don't** use rebase-merge or merge-commit strategies on `main`.
  Squash is the rule. (Configure GitHub repo settings to enforce.)
- **Don't** force-push to `main`. Force-push on your own feature
  branch is fine.
- **Don't** write a PR title that won't survive as a good squash commit
  subject. The title *is* the commit message.
