# Topic: Coding style & formatting (C# / .NET)

**Status:** `[~]` drafted — high-level stance + key specifics confirmed
2026-05-11. Remaining open: `.editorconfig` starter file, additional
analyzers / Roslyn rules.

**Scope:** C# language features, formatting, and analyzer conventions ASG
expects in every backend repo. Frontend coding style is a separate concern
(framework topic still open).

---

## Interview questions

Already captured (see Convention section):

- Modern C# features preference
- LTS version policy
- File-scoped namespaces
- Indentation / nesting reduction
- MS style guide as base reference

Still open:

1. `.editorconfig` — does ASG have a starter file? Or rely on the
   Microsoft default and only add ASG-specific overrides?
2. Analyzers / Roslyn rules — anything enforced beyond compiler defaults
   (e.g., StyleCop, Roslynator, custom rule set)?

---

## Convention

**Confirmed 2026-05-11 (high-level stance):**

- **Modern C# features are preferred** when they improve clarity. This
  includes (non-exhaustive): file-scoped namespaces, top-level statements
  where appropriate, `record` types for value-shaped data, primary
  constructors, target-typed `new()`, collection expressions (`[1, 2, 3]`),
  pattern matching and switch expressions, `nameof`, `is { ... }` property
  patterns, range/index operators, `using` declarations, `nullable`
  reference types. Use them; don't write 2010-era C# in 2026.
- **.NET LTS versions for new scaffolds.** New projects target the
  current .NET LTS release at the time of scaffolding. **As of 2026-05,
  that is .NET 10** (GA Nov 2025, supported through Nov 2028). Non-LTS /
  STS releases acceptable for experimentation or short-lived tooling, but
  not for production services.
- **Nullable reference types are enabled** in new projects
  (`<Nullable>enable</Nullable>` in the csproj or `<Nullable>enable</Nullable>`
  in `Directory.Build.props`). Annotate reference types accordingly;
  don't suppress nullability warnings unless there's a clear reason
  (and prefer `!` on a specific expression over project-wide suppression).
- **`TreatWarningsAsErrors` is disabled.** Warnings remain warnings;
  they don't break the build. Agents should still address compiler
  warnings rather than ignoring them, but the gate is the build
  succeeding, not warning-free output.
- **DTOs and value-shaped types use `record` by default**:
  `public sealed record CreateOrderRequest(Guid CustomerId, ...)`. Use
  `class` only when reference equality, inheritance, or mutability is
  genuinely required.
- **File-scoped namespaces** (`namespace Foo.Bar;`) — not block-scoped.
  One namespace per file (matches the file ↔ type rule).
- **Reduce indentation and nesting** when it doesn't harm readability:
  - Prefer early returns / guard clauses over deeply nested `if`.
  - Prefer expression-bodied members for trivial getters / one-liners.
  - Prefer switch expressions over `if/else if` chains when matching on a
    discriminator.
  - Prefer LINQ over manual loops for transforming collections — but not
    when the LINQ chain becomes less readable than the equivalent loop.
- **Microsoft's C# coding conventions are the base reference.** Defer to
  the official MS style guide whenever ASG hasn't explicitly diverged.
  Authoritative sources:
  - https://learn.microsoft.com/dotnet/csharp/fundamentals/coding-style/coding-conventions
  - https://learn.microsoft.com/dotnet/csharp/fundamentals/coding-style/identifier-names
  - .NET runtime's `CONTRIBUTING.md` style notes for canonical examples.

**Open / pending follow-up:** see Interview questions section.

## Rationale

- **Modern C# features over legacy idioms** because the language has moved
  forward dramatically since .NET Framework — refusing to use newer
  features means writing more code that does less. Agents working in ASG
  code should reach for `record`, switch expressions, pattern matching,
  etc., by default.
- **Nullable enabled, warnings-as-errors disabled** is a deliberate split:
  nullability annotations are valuable as design-time signals (and as
  documentation of intent), but breaking the build on every warning would
  slow iteration and surface noise in mixed-vintage code. The trade-off:
  agents must not let nullability warnings accumulate silently.
- **`record` by default** for DTOs gives value equality, immutability, and
  concise syntax for free. Use `class` only when you actually need
  reference identity or mutability — e.g., EF Core entities, long-lived
  service objects, types with behavior.
- **LTS-only for production** keeps support windows long enough that ASG
  isn't forced into urgent version migrations every 12 months. STS
  releases have shorter support; reserve them for tools and experiments.
- **File-scoped namespaces** save one indentation level on every file and
  match Roslyn's recommended modern style. There's no readability cost.
- **Reduce nesting** because deep nesting is the single biggest signal
  that a method is doing too much. Early returns make the happy path
  obvious; switch expressions linearize decision trees.
- **MS style as base** removes hundreds of micro-decisions. ASG only
  needs to document where it diverges, not re-derive every rule.

## Examples

**File-scoped namespace + modern features:**

```csharp
namespace ASG.Billing.Application.Orders;

public sealed record OrderDto(Guid Id, Guid CustomerId, decimal Total, OrderStatus Status);

public sealed class OrderService(IOrderRepository repository, IClock clock) : IOrderService
{
    public async Task<OrderDto?> GetAsync(Guid id, CancellationToken ct)
    {
        var order = await repository.FindAsync(id, ct);
        if (order is null) return null;

        return new OrderDto(order.Id, order.CustomerId, order.Total, order.Status);
    }

    public string DescribeStatus(OrderStatus status) => status switch
    {
        OrderStatus.Pending   => "Awaiting payment",
        OrderStatus.Paid      => "Payment received",
        OrderStatus.Shipped   => "On the way",
        OrderStatus.Cancelled => "Cancelled",
        _                     => "Unknown",
    };
}
```

Notice: file-scoped namespace, `record` for DTO, primary constructor on
the service, expression-bodied switch, early return.

**Guard-clause style (preferred):**

```csharp
public Result<Order> ApplyDiscount(Order order, decimal percent)
{
    if (order is null)             return Result.Fail("order required");
    if (percent <= 0)              return Result.Fail("percent must be positive");
    if (percent > 100)             return Result.Fail("percent cannot exceed 100");
    if (order.Status != Open)      return Result.Fail("order must be open");

    order.Total *= (1 - percent / 100m);
    return Result.Ok(order);
}
```

Not:

```csharp
public Result<Order> ApplyDiscount(Order order, decimal percent)
{
    if (order is not null)
    {
        if (percent > 0 && percent <= 100)
        {
            if (order.Status == Open)
            {
                order.Total *= (1 - percent / 100m);
                return Result.Ok(order);
            }
            else
            {
                return Result.Fail("order must be open");
            }
        }
        // ... etc.
    }
}
```

## Anti-patterns

- **Don't** use block-scoped namespaces (`namespace Foo.Bar { ... }`) in
  new code. Use file-scoped.
- **Don't** target non-LTS .NET versions for production services.
- **Don't** write deeply nested `if`/`else` when guard clauses or a switch
  expression would linearize the control flow.
- **Don't** refuse modern features out of "old-codebase consistency" —
  consistency with 2010 idioms is not a benefit. Update style in files
  you touch.
- **Don't** invent a parallel style guide. Defer to Microsoft's
  conventions; only document explicit ASG divergences.
- **Don't** target a non-LTS .NET version for a new production service.
  As of 2026-05, that means .NET 10.
- **Don't** suppress nullability project-wide (`<Nullable>disable</Nullable>`
  or `#nullable disable`) just to silence warnings. Address the warning,
  or suppress at the specific expression level with `!` when justified.
- **Don't** reach for `class` for a value-shaped DTO. Use `record`.
