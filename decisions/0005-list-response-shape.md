# 0005 — List / collection endpoint response shape

**Status:** Open — flagged important
**Date:** 2026-05-11
**Deciders:** Curtis Rutland (ASG)

## Context

Every list/collection endpoint (`GET /api/orders`, `GET /api/customers`,
etc.) needs a consistent response shape. ASG has not chosen one. Without
a convention, every list endpoint would shape its response slightly
differently, making client-side handling inconsistent and pagination
retrofits painful.

This decision is **important** — it affects every list endpoint in every
ASG backend repo, and changing it later is a breaking change to every
client consuming those endpoints.

## Options considered

### Option 1: Paginated envelope (always)

Every collection endpoint returns an object with `items` plus pagination
metadata, even if the endpoint isn't paginated yet:

```jsonc
{
  "items": [ { ... }, { ... } ],
  "totalCount": 42,
  "page": 1,
  "pageSize": 25,
  "hasNextPage": true
}
```

- **Pros**: Single shape across the entire API. Adding pagination
  later is non-breaking. Clients write one code path for all lists.
- **Cons**: Tiny over-the-wire bloat for unpaginated endpoints. Initial
  feel: "why is there pagination metadata for a 3-item list?"

### Option 2: Raw JSON array

Return `[ { ... }, { ... } ]` directly. Pagination, when needed, via
response headers (`X-Total-Count`, `Link: <...>; rel="next"`).

- **Pros**: Simpler payload; matches GitHub-style APIs. Body is the
  collection.
- **Cons**: Pagination metadata via headers is less discoverable to
  client developers. Two distinct paradigms (header-pagination vs
  body-pagination) coexist if any endpoint moves to envelope later —
  breaking change.

### Option 3: Envelope when paginated, raw array otherwise

Hybrid — small/static lists return raw arrays; paginated endpoints
return envelopes. Per-endpoint decision.

- **Pros**: No bloat on small lists.
- **Cons**: Inconsistent. Promoting a list to paginated becomes a
  breaking change. Clients must check the shape per endpoint.

### Option 4: Envelope-only, custom metadata key set

Like Option 1 but with a different metadata convention — e.g., wrap
under `data` instead of `items`, or include `_links` HATEOAS-style.

- **Pros**: Flexibility to encode cursors, links, etc.
- **Cons**: More design work; risk of overdesigning. Diverges further
  from common patterns.

## Decision

**Open** as of 2026-05-11.

**Recommended working default until decided**: Option 1 — paginated
envelope always. Specifically:

```jsonc
{
  "items": [ ... ],
  "totalCount": <int>,
  "page": <int>,         // 1-based
  "pageSize": <int>,
  "hasNextPage": <bool>
}
```

Rationale for the default: it's the conservative pick — adding
pagination later is the most-common evolution path for a list endpoint,
and Option 1 makes that evolution non-breaking. Options 2 and 3 risk
forcing a breaking change later.

Agents writing list endpoints today should use this shape. Single-
resource endpoints (`GET /api/orders/{id}`) return the resource
directly, not in an envelope — the envelope is *only* for collections.

## Consequences

- Until decided, list endpoints follow the working default. Pagination
  query params (e.g., `?page=1&pageSize=25`) are accepted even if
  the endpoint doesn't yet paginate; the server returns all results
  in `items` and reports the count.
- If Option 2 (raw array) is chosen later, every existing list endpoint
  becomes a breaking change for clients. This is the strongest
  argument for picking and committing to Option 1.
- Decision intersects with the error-envelope decision (`0002`).
  Error responses are a separate shape; this decision only governs
  successful list responses.
- Cursor-based pagination (opaque tokens) is **not** covered by this
  decision — it's a refinement that can be added later (e.g., a
  `nextCursor` field on the envelope) without breaking the basic shape.
