# Topic: Testing strategy

**Status:** `[ ]` not-started — **no decisions made yet.** Flagged 2026-05-11
during the project-structure interview. ASG has not yet decided on a testing
strategy; agents reading any template should treat this section as
*unstandardized* until this topic is filled in.

**Scope:** Unit / integration / E2E split, what to mock, test project
location and naming, test framework choice, code coverage expectations,
how tests are run locally and in CI.

---

## Interview questions

Backend (.NET):

1. Test framework — xUnit, NUnit, MSTest?
2. Assertion library — built-in, FluentAssertions, Shouldly?
3. Mocking library — Moq, NSubstitute, FakeItEasy, none?
4. Where do test projects live — sibling to src, separate `tests/` folder,
   both (unit close, integration separate)?
5. Naming pattern — `MethodName_Scenario_Expectation`? `Should_X_When_Y`?
6. What's mocked vs. not — the database, external HTTP, message bus,
   the clock?
7. Integration test approach — `WebApplicationFactory` + in-memory DB?
   Testcontainers? Real ephemeral DB?
8. Code coverage target / enforcement?
9. Snapshot / approval tests used anywhere?

Frontend (web):

1. Unit test framework — Vitest, Jest, framework-default?
2. Component test approach — Testing Library, Enzyme, framework-default?
3. E2E framework — Playwright, Cypress, none?
4. What's mocked at the frontend boundary — MSW, manual stubs, fixtures?

General:

1. What does "verify a change" mean before commit — build + test?
   build + test + lint?
2. CI requirements — must all tests pass on PR? Coverage gate?
3. Are flaky tests tolerated / quarantined / blocked-on?

---

## Convention

*Not yet decided.* Track decisions in `decisions/` when they're made.

## Rationale

*Not yet captured.*

## Examples

*Not yet captured.*

## Anti-patterns

*Not yet captured.*
