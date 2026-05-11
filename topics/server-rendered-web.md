# Topic: Server-rendered web (Razor Pages)

**Status:** `[~]` drafted — framework choice confirmed 2026-05-11
(**Razor Pages**, see `decisions/0007`). All other interview questions
are open.

Flagged 2026-05-11 when generating the example Razor CLAUDE.md
(`examples/CLAUDE.razor-pages.md`) surfaced two gaps that this topic
owns:

1. **Razor Pages vs MVC + Views** — answered 2026-05-11: **Razor
   Pages**. The api-design topic answered the API-endpoint question
   separately (MVC Controllers, not Minimal APIs); this topic answers
   the server-rendered HTML question.
2. **Razor-specific conventions** — page handlers, tag helpers,
   anti-forgery, layouts, partials, view components, model binding —
   none of which appear in the other topics. **Still open.**

**Scope:** Conventions for ASG .NET server-rendered web applications
(MPAs). Covers the choice of templating framework (Razor Pages vs
MVC + Views), route/page handler patterns, view templating helpers,
anti-forgery policy, layouts and partials, view components, and model
binding. Does **not** cover SPA frontends (separate topic, blocked on
the frontend framework decision).

---

## Interview questions

### Framework choice

**Confirmed (2026-05-11):** Razor Pages is the default for new ASG
server-rendered web apps. See `decisions/0007-razor-pages-vs-mvc-views.md`.

Remaining open:

1. Is there a size / complexity threshold where MVC + Views is preferred
   over Razor Pages (e.g., richer flows, complex routing)?
2. Is mixing Razor Pages and MVC controllers + views in the same
   project allowed, or do we standardize on Razor Pages per repo?
3. How does this interact with the API-design topic's choice of MVC
   Controllers for JSON endpoints — does a Razor Pages app that *also*
   exposes APIs mix Razor Pages + Controllers (controllers only for
   `/api/...`)? Working assumption: yes, mixing is fine when controllers
   are restricted to `/api/...`.

### Page / route handlers (mostly Razor Pages)

5. Handler naming — `OnGet` / `OnPost` / `OnGetAsync` / `OnPostAsync`?
   Default to async? Always require `CancellationToken`?
6. Named handlers (`OnPostExportAsync`, `OnGetEditAsync`) — used or
   avoided? Any naming convention for the handler name?
7. Where does the `PageModel` class live — colocated with the `.cshtml`
   (default `Foo.cshtml.cs`) or factored out? When (if ever) is
   factoring out justified?
8. When should logic move out of the `PageModel` into a service?
   PageModel as thin glue vs PageModel as business logic.

### View templating

9. **Tag helpers vs HTML helpers** — strict preference (e.g., "tag
   helpers only") or pragmatic mix?
10. Custom tag helpers — common pattern, or avoided in favor of
    partials / view components?
11. **View components** — used, avoided, or "only when partials
    aren't enough"? Naming convention?
12. Inline C# in views (`@{ ... }` blocks) — discouraged in favor of
    PageModel logic?

### Security

13. **Anti-forgery token policy** — `[ValidateAntiForgeryToken]` on
    every POST handler, or `AutoValidateAntiforgeryTokenAttribute`
    globally? Any exception cases?
14. Cookie auth scheme defaults — cookie name, sliding expiration,
    HTTPS-only, SameSite policy? Or defer to a future auth topic?
15. CSP / security headers — included by middleware? Which ones?

### Layouts, partials, view imports

16. Standard layout file name and location (default:
    `Pages/Shared/_Layout.cshtml`)? Per-section layouts allowed?
17. Partial naming convention — underscore prefix
    (`_PartialName.cshtml`)? Where do partials live?
18. `_ViewImports.cshtml` — at `Pages/` root only, or nested per
    feature? Standard `@addTagHelper` set?
19. `_ViewStart.cshtml` — used to set the default layout? Per-feature
    overrides allowed?

### Model binding

20. `[BindProperty]` always (on PageModel properties intended for
    binding), or selectively?
21. Explicit `[FromForm]` / `[FromQuery]` / `[FromRoute]` annotation
    preference vs relying on convention-based binding?
22. Custom model binders — established convention or per-need?

### Forms and UI

23. Validation pattern — DataAnnotations on PageModel properties,
    FluentValidation, both? (May defer to validation topic.)
24. Form post-redirect-get pattern enforced?
25. Client-side validation — built-in unobtrusive jQuery validation,
    or modern alternative?

### Frontend assets

26. CSS / JS bundling — `Microsoft.AspNetCore.WebOptimizer`,
    `BuildBundlerMinifier`, hand-rolled, none?
27. Static asset organization in `wwwroot/` — by feature, by type
    (`css/`, `js/`, `images/`)?
28. CSS framework — Bootstrap (default), Tailwind, custom?

### Other

29. TempData / session usage — any conventions or caveats?
30. Localization / globalization — supported by default? Resource
    file conventions?
31. SignalR or other real-time additions — convention or per-need?

---

## Convention

**Confirmed 2026-05-11:**

- **Framework: Razor Pages** for ASG server-rendered web apps. Not
  MVC + Views. Not Blazor (different stack — separate concern).
- Mixing with MVC Controllers is acceptable for the API surface
  (controllers handle `/api/...` JSON endpoints; Razor Pages handle
  server-rendered routes). Pure-MVC-with-Views for server-rendered
  HTML is not the ASG default.

**All other conventions for server-rendered web (page handler
patterns, tag helpers, anti-forgery, layouts, model binding, etc.) are
still open** — see Interview questions section above.

## Rationale

*To be filled in.*

## Examples

*To be filled in.*

## Anti-patterns

*To be filled in.*
