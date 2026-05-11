# 0003 — Exception → HTTP status code mapping convention

**Status:** Open
**Date:** 2026-05-11
**Deciders:** Curtis Rutland (ASG)

## Context

ASG backends throw exceptions for expected-but-erroneous conditions, and
a global `IExceptionHandler` catches and converts them to HTTP responses.
The question is *how* the handler decides which status code to emit for
which exception type. ASG has not standardized this.

Without a convention, each repo could map differently — agents writing
endpoints might pick 404, 400, or 422 for the same scenario depending on
which repo they're in, which makes client-side error handling
inconsistent across the ASG API surface.

## Options considered

### Option 1: Convention-based mapping (BCL type → status code)

A documented table from exception type to status code; the handler reads
the type and emits the corresponding status:

| Exception type                         | Status |
|----------------------------------------|--------|
| `KeyNotFoundException`                 | 404    |
| `ArgumentException` (+ subclasses)     | 400    |
| `ValidationException` / Fluent failures| 400 (with details) |
| `InvalidOperationException`            | 409    |
| `UnauthorizedAccessException`          | 401 or 403 (TBD) |
| `OperationCanceledException`           | 499 or no-response (TBD) |
| `NotImplementedException`              | 501    |
| Custom domain exceptions               | derive from one of the above, or carry their own value |
| Anything else                          | 500    |

- **Pros**: Predictable for agents — once they memorize the table they
  always know what status will come out. Centralized in one handler;
  one place to change.
- **Cons**: Some exceptions are ambiguous (`InvalidOperationException`
  → 409 isn't universal). Forces every custom exception to fit one of
  the buckets.

### Option 2: Exception carries its own status

Custom exceptions expose an `HttpStatusCode` property (or are tagged
with an attribute / marker interface):

```csharp
public sealed class OrderNotFoundException(Guid id)
    : DomainException(StatusCodes.Status404NotFound,
        $"Order {id} not found.");
```

The handler reads the value off the exception.

- **Pros**: Self-documenting at the throw site. New exceptions don't
  require touching the handler.
- **Cons**: Doesn't help for BCL exceptions (no status property). Need a
  fallback table anyway. ASG already prefers BCL types where they fit,
  which limits the impact.

### Option 3: Per-handler decision (one `IExceptionHandler` per exception type)

A separate handler for each exception family — `NotFoundExceptionHandler`,
`ValidationExceptionHandler`, `DefaultExceptionHandler`. Each knows the
right status.

- **Pros**: Strongly modular; each handler is small and testable. Easy
  to add a new family without modifying existing handlers.
- **Cons**: Handler count grows linearly with exception families.
  Mapping logic is spread across N files rather than one table.

## Decision

**Open** as of 2026-05-11. No ASG-wide convention chosen.

**Recommended working default until decided**: Option 1 (convention-based
table) with the mapping above. Agents writing new code should follow
the table; custom domain exceptions should derive their semantic
behavior from one of the listed BCL families (or extend the table when
landing this decision).

## Consequences

- Until decided, the table above is the working default. The
  `IExceptionHandler` lives in one place per repo and implements the
  table.
- If Option 2 (exception carries status) is chosen later, custom
  exception types get refactored to expose the value; the central
  handler simplifies. BCL exceptions still need a fallback table.
- If Option 3 (handler per type) is chosen later, the single handler
  splits into N. The mapping table becomes one handler per row.
- Decision intersects with `0002` (error envelope shape): once both are
  decided, agents have a complete spec — "when X exception is thrown,
  the response is HTTP status Y with body shape Z."
