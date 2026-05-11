# Topic: Logging & telemetry

**Status:** `[~]` drafted — spine confirmed 2026-05-11. Correlation ID
strategy, metrics conventions, and log retention policy remain open.

**Scope:** How ASG .NET backends emit log messages and operational
telemetry. Covers logging library choice, telemetry backend, log level
discipline, PII/secrets policy, structured-logging style, request/response
logging, and the boundary with `IExceptionHandler` (see error-handling
topic).

---

## Interview questions

Already captured (see Convention):

- Logging library / provider
- Telemetry / observability backend
- PII / sensitive data policy
- Catch-site vs global-handler logging
- Structured-logging style
- Production minimum log level
- Request/response logging approach

Still open:

1. Correlation / trace ID strategy — rely on `Activity.Current.TraceId`
   (auto-set by App Insights), surface a custom `X-Correlation-Id` header,
   or both?
2. Custom metrics — `System.Diagnostics.Metrics` API + App Insights, or
   App Insights `TelemetryClient.TrackMetric`? When should a counter or
   gauge be added?
3. Log retention / sampling policy — App Insights cost is real; what
   sampling rate is default for production?
4. Local-dev log destination — Console only? File? Seq?

---

## Convention

**Confirmed 2026-05-11:**

- **Logging library: built-in `ILogger<T>`** (`Microsoft.Extensions.Logging`)
  by default. Code injects `ILogger<T>` and uses the abstraction; no
  third-party logging library is added unless a client engagement
  requires it. **Escape hatch**: Serilog (behind `ILogger<T>`) is
  acceptable when a client requires detailed file-based logging that
  the built-in providers don't deliver well. Even then, application
  code keeps using `ILogger<T>` — only the registration changes.
- **Telemetry: Azure Application Insights via the direct SDK**
  (`Microsoft.ApplicationInsights.AspNetCore`). Matches ASG's
  Azure-preferred service stance (see build-and-run topic). Not
  OpenTelemetry — direct SDK is the default.
- **PII / secrets in logs: never.** No personal data (names, emails,
  phone numbers, addresses, government IDs), no secrets (tokens,
  passwords, keys, full connection strings), no full request or
  response bodies. Log identifiers (`{OrderId}`, `{CustomerId}`,
  `{TenantId}`) and non-PII context only. When in doubt, leave it out.
- **Catch-site logging: don't double-log.** The global
  `IExceptionHandler` is responsible for logging unhandled exceptions
  on their way out. Catch sites do **not** also log unless they're
  enriching with information the handler won't have — for example,
  domain context known at the catch site but not visible from the
  exception object. The default rule is: catch + rethrow without a
  log call.
- **Structured logging: always message templates with named placeholders.**
  Use `logger.LogInformation("Order {OrderId} created for {CustomerId}",
  orderId, customerId);` — the placeholders become structured fields
  in App Insights and any other sink. **Interpolated strings in log
  calls are an anti-pattern** because they collapse all properties
  into one flat message and lose structured-search capability.
- **Production minimum log level: Warning.** Anything Information or
  below is filtered out in production. Information-level logs are
  useful for local development and on-demand debug builds but should
  not be relied on appearing in production telemetry. This implies:
  - Use **Warning** for events that are genuinely concerning but not
    yet errors (degraded path taken, retry succeeded, slow operation).
  - Use **Error** for handled error conditions where the request failed.
  - Use **Critical** for failures that affect service health.
  - Use **Information** sparingly — only for events you'd be okay losing
    in production, or for local-dev breadcrumbs.
- **Request / response logging: off.** Application Insights
  auto-instruments inbound HTTP requests (URL, status, duration,
  dependencies). No custom request-logging middleware (`UseHttpLogging`,
  hand-rolled) — manual request logging risks accidentally logging PII
  in headers or bodies. Trust the auto-instrumentation.

**Open / pending follow-up:** see Interview questions section.

## Rationale

- **Built-in `ILogger<T>` as default** keeps the dependency surface
  small and avoids the licensing/maintenance overhead of a third-party
  library when ASG's logging needs are met by the BCL. The
  abstraction means swapping providers later is one-line in
  `Program.cs`. Serilog is reserved for the specific case where a
  client contract requires detailed file output (audit logs, on-prem
  file-based shipping, etc.).
- **App Insights direct SDK over OpenTelemetry** is the pragmatic
  ASG choice given Azure-preferred services. OTel adds an abstraction
  layer for vendor-neutrality, but ASG isn't planning to swap backends.
  Direct SDK gives best out-of-box auto-instrumentation and zero-config
  dependency tracking.
- **Strict no-PII** policy avoids GDPR / privacy compliance landmines.
  Logs are easy to forget about and easy to leak; the safest stance is
  "PII never lives here." Identifiers (GUIDs, internal IDs) carry the
  context agents need for debugging without exposing personal data.
- **Catch-site silence + handler logging** avoids the common pattern
  of logging the same exception 3 times as it bubbles. The
  `IExceptionHandler` has full request context (route, status, trace
  ID) and is the single source of truth for "this exception escaped."
  Catch sites only log when they're adding signal — usually because
  they're wrapping an exception and the original context would be lost.
- **Message templates** are the single most important modern logging
  rule. Without named placeholders, App Insights treats every log
  message as a unique string and structured search breaks. The cost
  of typing `{OrderId}` over `$orderId` is trivial; the benefit is
  every prod log being queryable.
- **Warning-level production filter** reduces both noise and App
  Insights cost. Information logs are by definition "things that
  happened normally" — they have low diagnostic value at scale and
  high volume. Production observability comes from auto-instrumented
  requests + Warning-and-above signals from application code.
- **No request-logging middleware** is a direct consequence of the
  PII policy: any request-body logging risks logging the wrong thing.
  App Insights auto-instrumentation captures the metadata (URL,
  status, duration) without ever touching the body.

## Examples

**Logger injection in a controller (primary-constructor style):**

```csharp
[ApiController]
[Route("api/orders")]
public sealed class OrdersController(
    IOrderService orders,
    ILogger<OrdersController> logger) : ControllerBase
{
    [HttpPost]
    public async Task<ActionResult<CreateOrderResponse>> CreateAsync(
        CreateOrderRequest request, CancellationToken ct)
    {
        var created = await orders.CreateAsync(request, ct);

        // Note: structured fields, no PII, Information level (won't surface in prod).
        logger.LogInformation(
            "Order {OrderId} created (total {Total:C}, items {ItemCount})",
            created.Id, created.Total, request.Items.Count);

        return Ok(new CreateOrderResponse(created.Id, created.Total));
    }
}
```

**Warning at a degraded-path branch:**

```csharp
public async Task<Quote> GetQuoteAsync(Guid productId, CancellationToken ct)
{
    try
    {
        return await pricingService.GetLiveAsync(productId, ct);
    }
    catch (HttpRequestException ex)
    {
        // Pricing service is down; fall back to cached pricing.
        // This is a real signal in prod, so Warning is the right level.
        logger.LogWarning(ex,
            "Live pricing unavailable for {ProductId}; falling back to cache.",
            productId);
        return await cache.GetQuoteAsync(productId, ct);
    }
}
```

**Catch-site that adds context before rethrowing (the rare valid case):**

```csharp
try
{
    await billingProvider.ChargeAsync(orderId, amount, ct);
}
catch (HttpRequestException ex)
{
    // The exception has no visibility into orderId. Wrap and rethrow;
    // the global handler will log the wrapped exception with the OrderId in context.
    throw new BillingProviderException(
        $"Failed to charge order {orderId} for {amount:C}", ex);
}
```

**Program.cs telemetry registration:**

```csharp
builder.Services.AddApplicationInsightsTelemetry();

// Production: filter our code at Warning, framework at Warning (already default).
builder.Logging.AddFilter("ASG", LogLevel.Warning);
```

**App Insights query example** (what a structured log enables):

```kusto
traces
| where customDimensions.OrderId == "f7a8b1e3-..."
| order by timestamp desc
```

## Anti-patterns

- **Don't** use interpolated strings in log calls:
  `logger.LogInformation($"Order {orderId} created");` ❌. The
  property is lost; every log becomes a unique unsearchable string.
- **Don't** log PII or secrets. No names, emails, phone numbers,
  addresses, government IDs, tokens, passwords, keys, full URLs with
  query-string secrets, or full request/response bodies.
- **Don't** log + rethrow at every catch site. The global handler
  already logs the exception with full request context.
- **Don't** install Serilog (or any other logging library) by default.
  Use built-in `ILogger<T>`. Add a third-party provider only when a
  client requires it.
- **Don't** install OpenTelemetry for new repos. App Insights direct
  SDK is the default.
- **Don't** enable `UseHttpLogging` or write a custom request-logging
  middleware. App Insights auto-instrumentation covers this; manual
  request logging risks PII exposure.
- **Don't** assume Information-level logs will be available in
  production. They're filtered out. Use Warning for anything you'd
  actually want to see post-incident.
- **Don't** call `logger.LogError` for an exception that's about to be
  rethrown — `throw` it; the handler logs.
