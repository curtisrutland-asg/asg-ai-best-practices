# Topic: Error handling

**Status:** `[~]` drafted — exception philosophy + handler approach
confirmed 2026-05-11. Error envelope shape and exception→status mapping
remain open decisions (see `decisions/0002` and `decisions/0003`).

**Scope:** How ASG .NET backends raise, propagate, catch, and surface
errors. Covers exception philosophy, custom exception types, global
exception middleware, and re-throw conventions. Does **not** cover
validation as a separate concern (that's a future topic), or logging
at catch site (logging topic, not yet drafted).

---

## Interview questions

Already captured (see Convention):

- Throw vs return philosophy
- Custom exception type policy
- Global error middleware approach
- Re-throw rules

Still open (logged as ADRs):

1. API error envelope shape — ProblemDetails vs custom vs minimal.
   See `decisions/0002-api-error-envelope.md`.
2. Standard exception→HTTP status code mapping. See
   `decisions/0003-exception-status-code-mapping.md`.

Other questions to revisit when logging / validation topics open:

3. Should catch sites log, or rely on the global handler?
4. `OperationCanceledException` / cancellation token handling — surface
   as which status code?
5. How are validation failures represented (FluentValidation,
   DataAnnotations, custom)? Probably a separate topic.

---

## Convention

**Confirmed 2026-05-11:**

- **Throw, don't return** — traditional .NET style. ASG does **not**
  adopt `Result<T>` / `OneOf<T, Error>` / Either-style return types as
  a baseline. Expected-but-erroneous conditions (not-found, business
  rule violations, unauthorized access, validation failures) are
  represented as exceptions, not as branches in the return type.
- **BCL exception types first; small custom domain set when needed.**
  Use `ArgumentException`, `ArgumentNullException`, `KeyNotFoundException`,
  `InvalidOperationException`, `UnauthorizedAccessException`,
  `NotSupportedException`, etc. where they fit. Define a custom type
  (e.g., `OrderNotFoundException`, `InsufficientFundsException`) only
  when it represents a genuine domain concept that justifies the type
  name. Custom types are conventionally derived from `Exception`
  directly unless ASG later defines a common base.
- **Global exception handling: `IExceptionHandler`** (the .NET 8+ pattern).
  Register handlers with `services.AddExceptionHandler<TYourHandler>()`
  and add `app.UseExceptionHandler()` to the pipeline. Multiple handlers
  can be chained — each handler returns `true` if it handled the
  exception, `false` to fall through to the next.
- **Re-throw rules**:
  - **Always preserve stack: use `throw;`** (bare) — never `throw ex;`.
    `throw ex;` resets the stack to the catch site and destroys the
    original call site information. This is a universal .NET rule, not
    an ASG preference.
  - **Wrap with new exception when adding context**: `throw new
    BillingProviderException("Failed to charge order " + orderId, ex);`
    The original `ex` becomes `InnerException`; the new exception adds
    semantic meaning for upstream handlers.
  - Choose between `throw;` and wrap based on **intent**: if you have
    nothing meaningful to add, preserve the stack. If you're adding
    domain context that helps the caller, wrap.

**Open / pending follow-up:**

- **Error envelope** (the response body shape when an exception becomes
  an HTTP response): not yet decided — see `decisions/0002`. Until
  decided, default to `ProblemDetails` because it's the ASP.NET Core
  built-in (`Results.Problem()`, `ProblemDetailsFactory`) and integrates
  natively with `IExceptionHandler`.
- **Exception → status code mapping**: not yet decided — see
  `decisions/0003`. Until decided, agents pick reasonable status codes
  per case (404 for not-found, 400 for bad input, 409 for conflict,
  401/403 for auth, 500 for unexpected).

## Rationale

- **Throw, don't return** matches the BCL's own conventions and most .NET
  code in the wild. Adopting Result types would diverge from the
  ecosystem and force agents to learn an ASG-specific abstraction on top
  of standard C#. The cost is that "errors as control flow" can mask
  signal in logs — but the global handler centralizes the formatting
  story, so callers don't have to think about it.
- **BCL types where they fit** avoids inventing parallel exception
  hierarchies. `KeyNotFoundException` already means what
  `OrderNotFoundException` would mean for many endpoints. Reserve
  custom types for cases where the type *name itself* carries
  domain meaning the BCL can't (`InsufficientFundsException` is more
  precise than `InvalidOperationException`).
- **`IExceptionHandler`** is the current Microsoft-recommended pattern,
  introduced in .NET 8. Handlers are DI-friendly, unit-testable
  in isolation, and composable (chain handlers for different exception
  families). Older `UseExceptionHandler("/error")` and MVC
  `ExceptionFilter` are legacy paths.
- **Re-throw discipline**: `throw ex;` is a long-standing C# footgun —
  it silently destroys the stack. The choice between `throw;` and
  wrapping is about whether you have something meaningful to add. Both
  are accepted; the rule is *don't lose information*.

## Examples

**Throwing an expected-but-erroneous condition:**

```csharp
public async Task<Order> GetAsync(Guid id, CancellationToken ct)
{
    var order = await repository.FindAsync(id, ct);
    if (order is null)
        throw new KeyNotFoundException($"Order {id} not found.");
    return order;
}
```

Or with a domain-specific type, when the name adds value:

```csharp
public async Task ApplyChargeAsync(Guid orderId, decimal amount, CancellationToken ct)
{
    var account = await accounts.GetAsync(orderId, ct);
    if (account.Balance < amount)
        throw new InsufficientFundsException(orderId, requested: amount, available: account.Balance);

    // ...
}
```

**Re-throw preserving stack:**

```csharp
try
{
    await client.SendAsync(message, ct);
}
catch (SqlException ex) when (ex.Number == 2627)  // duplicate key
{
    // We expected this; let the caller handle it.
    throw;
}
```

**Wrap to add context:**

```csharp
try
{
    await billingProvider.ChargeAsync(orderId, amount, ct);
}
catch (HttpRequestException ex)
{
    throw new BillingProviderException(
        $"Failed to charge order {orderId} for {amount:C}.", ex);
}
```

**`IExceptionHandler` skeleton:**

```csharp
public sealed class NotFoundExceptionHandler(ILogger<NotFoundExceptionHandler> logger) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext, Exception exception, CancellationToken ct)
    {
        if (exception is not KeyNotFoundException) return false;

        logger.LogInformation(exception, "Resource not found: {Message}", exception.Message);
        httpContext.Response.StatusCode = StatusCodes.Status404NotFound;
        await httpContext.Response.WriteAsJsonAsync(new ProblemDetails
        {
            Status = StatusCodes.Status404NotFound,
            Title = "Resource not found",
            Detail = exception.Message,
        }, ct);
        return true;
    }
}

// Program.cs
builder.Services.AddExceptionHandler<NotFoundExceptionHandler>();
builder.Services.AddExceptionHandler<DefaultExceptionHandler>(); // catch-all, last
builder.Services.AddProblemDetails();

var app = builder.Build();
app.UseExceptionHandler();
```

## Anti-patterns

- **Don't** use `throw ex;` ever. It clobbers the stack trace. Use
  `throw;` to preserve, or wrap to add context.
- **Don't** introduce `Result<T>` / `OneOf<T, Error>` / Either types as
  a baseline. ASG throws.
- **Don't** define a custom exception when a BCL type fits. Don't write
  `MyArgumentException` — use `ArgumentException`.
- **Don't** catch `Exception` broadly without either re-throwing,
  wrapping, or returning structured info. A silent `catch { }` is a bug.
- **Don't** rely on `UseExceptionHandler("/error")` or MVC
  `ExceptionFilter` for new code. Use `IExceptionHandler`.
- **Don't** put response-shaping logic in service-layer code. Services
  throw; the global handler formats the response. Keep the
  service ↔ HTTP boundary clean.
- **Don't** swallow `OperationCanceledException` — let it propagate so
  the caller knows the request was cancelled.
