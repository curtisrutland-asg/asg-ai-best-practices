# 0002 — API error envelope shape

**Status:** Open
**Date:** 2026-05-11
**Deciders:** Curtis Rutland (ASG)

## Context

When an exception (or expected business failure) becomes an HTTP response,
the response body needs a consistent shape so client code can parse errors
reliably. ASG has not chosen one. This decision affects every API endpoint
across every ASG backend repo.

The error-handling topic confirmed:

- Backends throw exceptions (not Result types).
- Global handler uses `IExceptionHandler` (.NET 8+ pattern).
- BCL exception types preferred + a few custom domain ones.

The remaining question is what the *response body* looks like when those
exceptions surface as HTTP errors.

## Options considered

### Option 1: RFC 7807 `ProblemDetails` (`application/problem+json`)

- **Pros**: Built into ASP.NET Core (`Results.Problem()`,
  `ProblemDetailsFactory`, `AddProblemDetails()`). Integrates natively
  with `IExceptionHandler`. Well-known internet standard — clients can
  reuse off-the-shelf parsers. Extensible via additional properties.
- **Cons**: The standard field names (`type`, `title`, `status`, `detail`,
  `instance`) take some getting used to. Less obviously self-documenting
  for clients new to RFC 7807.

### Option 2: Custom envelope (`{ error: { code, message, details } }`)

- **Pros**: Field names can be whatever ASG wants — `error.code`,
  `error.message`, `error.details`, etc. Easy for clients to understand
  at first glance. Can carry domain-specific structure (e.g.,
  `validationErrors: [{ field, message }]`).
- **Cons**: ASG reinvents a standard that already exists. Every client
  needs to learn the ASG-specific shape. No off-the-shelf tooling.
  Requires consistent enforcement across every repo / endpoint to avoid
  drift.

### Option 3: Status code + minimal/plain body

- **Pros**: Simplest possible. HTTP status code carries the meaning;
  body is empty or unstructured text.
- **Cons**: Almost always insufficient — clients need more than a 400
  to render a useful message to the end user. Validation errors in
  particular need per-field detail that won't fit in a status code.

## Decision

**Open** as of 2026-05-11. No ASG-wide choice has been made.

**Recommended working default until decided**: Option 1 (`ProblemDetails`).
Rationale: it's the path of least resistance — built into ASP.NET Core,
integrates with `IExceptionHandler` out of the box, and is a recognized
internet standard. Choosing it now defers no work; choosing it later
would require rewriting endpoints.

New repos created today should default to `ProblemDetails` unless the
repo's owner has a specific reason to do otherwise. This decision will
be revisited once ASG accumulates experience with the default.

## Consequences

- Agents writing new endpoints should emit `ProblemDetails`-shaped
  responses for errors until this decision is finalized. Use
  `Results.Problem()` in minimal APIs and `ProblemDetailsFactory` in
  MVC controllers.
- If ASG later picks Option 2 (custom envelope), existing endpoints
  using `ProblemDetails` will need a migration path. The
  `IExceptionHandler` pattern keeps that migration centralized to the
  handler layer — service code doesn't change.
- Validation error shape is a sub-question: if ProblemDetails is the
  envelope, validation failures use the standard `ValidationProblemDetails`
  type. If a custom envelope is chosen, the validation shape becomes
  part of that envelope's spec.
