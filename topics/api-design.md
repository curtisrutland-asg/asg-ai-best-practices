# Topic: API design

**Status:** `[~]` drafted — spine confirmed 2026-05-11. List response
shape is an open important decision (see `decisions/0005`). Error
response shape inherits from the error-handling topic's open decisions
(`decisions/0002`, `decisions/0003`).

**Scope:** REST conventions for ASG .NET backend APIs — endpoint style,
URL structure, HTTP method usage, status codes, versioning, JSON shape.
Does **not** cover validation (future topic), auth (future topic), or
caching/ETag (future topic).

---

## Interview questions

Already captured (see Convention):

- Endpoint definition style (Controllers vs Minimal APIs)
- JSON property naming policy
- Versioning strategy + specific header form
- HTTP method usage (PUT vs PATCH)
- Success status code conventions

Still open:

1. List / collection response shape — envelope vs raw array vs hybrid.
   **Flagged as an important decision.** See `decisions/0005`.
2. OpenAPI / Swagger generation — required for all repos? Which package
   (`Swashbuckle`, .NET 9's built-in `Microsoft.AspNetCore.OpenApi`)?
3. Idempotency keys (`Idempotency-Key` header) — used on POST endpoints?
4. ETag / caching headers — used or skipped?
5. Date/time wire format — ISO 8601 with offset (`2026-05-11T14:23:00-07:00`)?
   UTC only (`...Z`)?
6. Filter / sort / search query syntax — standard convention?
7. Bulk operations — separate endpoints, batch arrays, JSON Patch?

---

## Convention

**Confirmed 2026-05-11:**

- **Endpoint definition: MVC Controllers**, not Minimal APIs. Use
  `[ApiController]` + attribute routing on controller classes. One
  controller per resource (or closely-related resource group).
  Controllers live in `Controllers/` inside the API project.
- **JSON property naming: camelCase** in both requests and responses
  (`orderId`, `customerName`). This is the ASP.NET Core default since
  .NET 5; don't override `JsonSerializerOptions.PropertyNamingPolicy`.
- **URL casing: kebab-case, lowercase.** `/api/order-items/{id}` — see
  the naming-conventions topic for details.
- **DTO naming at the API boundary**: `<Operation>Request` /
  `<Operation>Response` per endpoint (e.g., `CreateOrderRequest`,
  `CreateOrderResponse`). `Dto` suffix is reserved for internal
  layer-crossing types. See naming-conventions topic.
- **Versioning: header-based via `X-Api-Version`.** Clients send
  `X-Api-Version: 2`. URL does **not** carry the version segment —
  routes are `/api/orders`, not `/api/v2/orders`. Implement via the
  `Asp.Versioning.Mvc` package (`AddApiVersioning(options => { ... })`)
  with `ApiVersionReader.Combine(new HeaderApiVersionReader("X-Api-Version"))`.
- **HTTP methods**:
  - `GET` — retrieve. Safe and idempotent.
  - `POST` — create.
  - **`PUT` — update (both full replacement and partial).** ASG does
    not use `PATCH`. A PUT body may contain all fields or only the
    fields the caller wants to update.
  - `DELETE` — remove.
- **Success status codes** (ASG-specific divergence from strict REST):
  - `GET` → **200 OK** with body.
  - `POST` (create) → **200 OK** with the created resource in the body.
    Do **not** return 201 Created. The `Location` header is optional.
  - `PUT` → **200 OK** with the updated resource in the body.
  - `DELETE` → **204 No Content** (no body).
  - Async work accepted but not complete → **202 Accepted** with a
    location/status URL.
- **Error responses** depend on two pending decisions:
  - Error envelope shape: `decisions/0002` (working default:
    `ProblemDetails`).
  - Exception → status code mapping: `decisions/0003` (working default:
    BCL convention-based table).
  - See the error-handling topic for `IExceptionHandler` plumbing.

**Open / pending follow-up:**

- **List response shape** (decision 0005): envelope with pagination
  metadata vs raw array vs hybrid. Important — affects every
  list endpoint. Working default until decided: **always envelope with
  pagination metadata**, even when not paginated, so the shape stays
  consistent and clients don't have to special-case "paginated vs not."

## Rationale

- **MVC Controllers over Minimal APIs**: Controllers give a clear
  per-resource structure (one file per resource), benefit from attribute
  routing + filters + model binding for free, and are easier to
  organize as APIs grow. Minimal APIs shine for tiny services but get
  noisy at scale — ASG's API surface is closer to "growing service"
  than "5-endpoint microservice." Note: this is a deliberate departure
  from the modern-default; the coding-style preference for modern
  C# features applies inside controllers (records for DTOs, primary
  constructors, etc.).
- **camelCase JSON** is the de facto standard for HTTP APIs consumed
  by browsers and JS/TS clients, which is most of them. It's also
  the ASP.NET Core default, so "don't override the default" is the
  rule.
- **Header versioning over URL versioning**: URLs name resources, not
  API versions. Putting `v2` in the URL makes every old URL go stale on
  a version bump and conflates resource identity with API contract.
  `X-Api-Version` keeps URLs stable and version negotiation
  out-of-band. Trade-off: harder for casual consumers (curl users) to
  hit the right version — they must remember to send the header.
- **PUT for both replacement and partial** trades REST purity for
  simplicity. Distinguishing PUT (replace) from PATCH (partial) doubles
  the per-endpoint design surface for marginal client benefit. ASG
  endpoints treat PUT as "update with whatever fields you send"; the
  server applies them. Clients that need full-replacement semantics
  send all fields.
- **200 OK with body for create** (not 201): Most callers want the
  created resource back immediately so they can show it / update local
  state. Forcing them to follow the `Location` header or re-fetch by
  ID is friction. 201 + `Location` is the REST-pure answer, but 200 +
  body is what consumers actually use.
- **204 on DELETE** because there's nothing meaningful to return; the
  resource is gone.

## Examples

**Controller skeleton:**

```csharp
namespace ASG.Billing.Api.Controllers;

[ApiController]
[Route("api/orders")]
public sealed class OrdersController(IOrderService orders) : ControllerBase
{
    [HttpGet("{id:guid}")]
    public async Task<ActionResult<OrderResponse>> GetAsync(Guid id, CancellationToken ct)
    {
        var order = await orders.GetAsync(id, ct);
        return Ok(OrderResponse.From(order));
    }

    [HttpPost]
    public async Task<ActionResult<CreateOrderResponse>> CreateAsync(
        CreateOrderRequest request, CancellationToken ct)
    {
        var created = await orders.CreateAsync(request, ct);
        return Ok(new CreateOrderResponse(created.Id, created.Total));  // 200, not 201
    }

    [HttpPut("{id:guid}")]
    public async Task<ActionResult<OrderResponse>> UpdateAsync(
        Guid id, UpdateOrderRequest request, CancellationToken ct)
    {
        var updated = await orders.UpdateAsync(id, request, ct);
        return Ok(OrderResponse.From(updated));
    }

    [HttpDelete("{id:guid}")]
    public async Task<IActionResult> DeleteAsync(Guid id, CancellationToken ct)
    {
        await orders.DeleteAsync(id, ct);
        return NoContent();  // 204
    }
}
```

**API versioning setup (Program.cs):**

```csharp
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = new HeaderApiVersionReader("X-Api-Version");
}).AddMvc();
```

**Controller with explicit version:**

```csharp
[ApiController]
[ApiVersion(2.0)]
[Route("api/orders")]  // no v2 segment in the URL
public sealed class OrdersV2Controller(IOrderService orders) : ControllerBase
{
    // ...
}
```

**JSON request/response (camelCase):**

```jsonc
// POST /api/orders   (X-Api-Version: 2)
// Request:
{
  "customerId": "0c0e1c4d-1d52-4f8a-b7d7-7e3b9b4c1234",
  "items": [
    { "sku": "WIDGET-001", "quantity": 3 }
  ]
}

// Response: 200 OK
{
  "orderId": "f7a8b1e3-...",
  "total": 29.97
}
```

## Anti-patterns

- **Don't** use Minimal APIs for new endpoints. ASG standardizes on
  MVC Controllers.
- **Don't** put `v1`, `v2`, etc. in the URL path. Use the
  `X-Api-Version` header.
- **Don't** override `PropertyNamingPolicy` to PascalCase or
  snake_case. camelCase is the rule.
- **Don't** use `PATCH`. `PUT` handles both full and partial updates.
- **Don't** return `201 Created` from POST endpoints. Return `200 OK`
  with the created resource in the body.
- **Don't** use `PascalCase` or `camelCase` in URL paths — they're
  kebab-case (`/api/order-items`, not `/api/orderItems`).
- **Don't** wrap successful responses in a `{ data: ... }` envelope
  unless required by the list-shape decision (0005). Single-resource
  responses are the resource directly.
- **Don't** define a controller with route `/orders` (missing the
  `/api` prefix). All ASG controllers live under `/api/...`.
