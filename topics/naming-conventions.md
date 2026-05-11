# Topic: Naming conventions

**Status:** `[~]` drafted — backend type/file/route conventions confirmed
2026-05-11. Frontend naming deferred until framework topic is settled.
Source-control naming (branches, commits, PRs) deferred to its own topic.

**Scope:** Names for files, types, namespaces, branches, commits, and
anything else where ASG enforces a pattern that an agent should follow
without being told twice.

---

## Interview questions

Backend (.NET / C#):

1. Namespace pattern — `ASG.<Product>.<Layer>`? `Company.Product.Module`?
   Match folder structure or independent?
2. Class naming — standard .NET PascalCase, but any ASG-specific suffix
   conventions (`Service`, `Manager`, `Handler`, `Helper`)?
3. Interface naming — `I`-prefix always, or only when there's a concrete
   counterpart?
4. Async method naming — `Async` suffix always, never, or only when
   ambiguous?
5. DTO / request / response naming — `FooRequest` / `FooResponse`?
   `FooDto`? `FooViewModel`?
6. File naming — one type per file? Match filename to type name?
7. Test naming pattern — `MethodName_Scenario_Expectation`?
   `Should_X_When_Y`? Class-per-method or class-per-SUT?

Frontend (web):

1. Component file naming — `PascalCase.tsx` (React) / `kebab-case.component.ts`
   (Angular) / framework default?
2. Hook naming (if React) — `useFoo` always, file `useFoo.ts`?
3. Where do types live — colocated `.types.ts`, central `types/` folder,
   inline?
4. Constants — `SCREAMING_SNAKE` for true constants, or all `camelCase`?

Source control:

1. Branch naming pattern — `feature/<ticket>-<slug>`? `<user>/<topic>`?
2. Commit message format — Conventional Commits (`feat:`, `fix:`)?
   Free-form? Ticket-prefixed?
3. PR title format — match commit format, or different rules?

General:

1. Anything ASG names *differently* from common .NET / web conventions?
   (Easy to miss — agents default to community norms.)
2. Casing in URLs / API routes — `kebab-case`, `camelCase`, `PascalCase`?

---

## Convention

**Backend (.NET / C#) — confirmed 2026-05-11:**

- **Namespaces** match `.csproj` name + folder structure (default .NET
  behavior). A file at `Controllers/OrderController.cs` in the
  `ASG.Billing.Api` project gets namespace `ASG.Billing.Api.Controllers`.
- **Project (`.csproj`) names** follow `ASG.<Product>.<Layer>` — captured
  in the project-structure topic.
- **Interfaces** always use the `I`-prefix: `IOrderService`, `IRepository`,
  `IClock`. No exceptions.
- **Async methods** get the `Async` suffix **only when a sync counterpart
  exists**. If only the async version is available, the suffix is
  optional. (Note: this diverges from the BCL's "always suffix" rule —
  apply ASG's version inside ASG repos.)
- **DTO / contract type naming** depends on the layer:
  - **API boundary**: `<Operation>Request` / `<Operation>Response` —
    e.g., `CreateOrderRequest`, `CreateOrderResponse`. Per-endpoint pair.
  - **Application ↔ infrastructure**: `<Concept>Dto` — e.g., `OrderDto`.
    Generic when direction doesn't matter.
  - `ViewModel` suffix is reserved for MVC/Razor view models (server-rendered).
- **File-to-type relationship**: mostly strict — one public type per file,
  filename matches the type name. Tiny closely-related types (small
  records, enums, helper structs) may be grouped into the primary type's
  file when grouping is clearer than separating.
- **Class suffixes** (`Service`, `Manager`, `Handler`, `Helper`, etc.) are
  **loose convention**, not enforced. Pick the most descriptive name;
  don't assume a suffix carries a defined architectural role unless the
  repo's existing code makes that explicit. Avoid `Helper` / `Utility` /
  `Manager` when a more specific name fits.
- **URL / API route casing**: `kebab-case`, lowercase. Example:
  `/api/order-items/{id}`. Not `camelCase`, not `PascalCase`.

**Frontend naming**: deferred. Will be captured once framework choice
(React / Angular / Blazor) is decided.

**Source-control naming** (branches, commits, PR titles): deferred to
the source-control topic.

## Rationale

- **Namespace = csproj + folders** lets agents derive the right
  `using`/`namespace` line just from the file path; no separate convention
  to memorize.
- **`Async` suffix only when paired** keeps method names shorter when
  there's no ambiguity. Adding the suffix to every async-returning method
  is noise when there's no sync version to disambiguate from. (The BCL
  applies "always suffix" because it exposes both sync and async pairs in
  many places — internal ASG code rarely needs both.)
- **`I`-prefix on interfaces** is the .NET community default and matches
  Visual Studio / Rider tooling expectations (e.g., DI registration
  scanners that look for `I*` ↔ `*` pairs).
- **Layered DTO naming** (Request/Response at API, Dto at boundary) lets
  the type name reveal which architectural layer a type belongs to,
  reducing cross-layer leakage.
- **Kebab-case routes** avoid case-sensitivity bugs (some web servers /
  proxies normalize URLs differently) and read more naturally in browser
  address bars and logs.
- **Loose class suffixes** acknowledge ASG hasn't standardized roles like
  "Service" or "Handler" — locking them down would force retrofitting
  existing code.

## Examples

**File → type → namespace pairing:**

```
src/ASG.Billing.Api/Controllers/OrderController.cs
  namespace ASG.Billing.Api.Controllers;
  public sealed class OrderController : ControllerBase { ... }
```

```
src/ASG.Billing.Application/Orders/IOrderService.cs
  namespace ASG.Billing.Application.Orders;
  public interface IOrderService { ... }
```

**Async naming:**

```csharp
// Both sync and async exist → suffix required on async one
public Order GetOrder(Guid id) { ... }
public Task<Order> GetOrderAsync(Guid id, CancellationToken ct) { ... }

// Only async exists → suffix optional
public Task<Order> GetOrder(Guid id, CancellationToken ct) { ... }
```

**DTO naming by layer:**

```csharp
// API boundary — per endpoint
public sealed record CreateOrderRequest(Guid CustomerId, List<LineItem> Items);
public sealed record CreateOrderResponse(Guid OrderId, decimal Total);

// Application ↔ infrastructure
public sealed record OrderDto(Guid Id, Guid CustomerId, decimal Total, OrderStatus Status);
```

**Routes:**

```
GET    /api/orders                       ✓
GET    /api/orders/{id}                  ✓
GET    /api/orders/{id}/line-items       ✓ (kebab-case for multi-word segment)
POST   /api/customers/{id}/billing-info  ✓
```

## Anti-patterns

- **Don't** drop the `I`-prefix from interfaces.
- **Don't** suffix every async method with `Async` reflexively when no
  sync counterpart exists — ASG's rule diverges from the BCL's.
- **Don't** use `Dto` at the API boundary. Use `<Operation>Request` /
  `<Operation>Response`. Reserve `Dto` for internal layer-crossing types.
- **Don't** use `camelCase` or `PascalCase` in route paths
  (`/api/orderItems` ❌; use `/api/order-items` ✓).
- **Don't** invent a new namespace scheme — match the csproj + folder
  structure default. If an existing file violates this, fix it as you
  pass through rather than spreading the deviation.
- **Don't** reach for `Helper`, `Manager`, or `Utility` suffixes when a
  domain-specific name would describe the type's role better.
