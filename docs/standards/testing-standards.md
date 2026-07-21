---
doc_type: standard
service: null
status: accepted
layer: testing
applies_to_agents: [coding-agent, testing-agent, code-review-agent]
---

# Testing Standards

Physical layout (`tests/UnitTests`, `IntegrationTests`, `ContractTests`) is already fixed by [`folder-structure.md`](folder-structure.md). This file owns what belongs in each, how much, and what "passing" means.

## Test Pyramid

- **Unit tests** — the bulk of the suite. One aggregate/handler/value-object per test class, no I/O (no real DB, broker, HTTP call, or clock). Colocated by feature, mirroring `Application/Features`.
- **Integration tests** — a real (or containerized, e.g. Testcontainers) database/broker, still inside one service. Verify a vertical slice end-to-end within its own boundary, including the Outbox write.
- **Contract tests** — consumer-driven, run against the schemas published to `kart-shared` (or the consuming project's equivalent shared-contracts package). Verify this service's published events/API match what it claims to publish, and that its consumed events/APIs match what it actually depends on — this is what the Contract Compatibility Agent checks before allowing a breaking change.
- **E2E tests** — owned by the Testing Agent in staging (post-deploy), not by the service repo itself. A service repo's own CI never spins up other services to test against.

## What to Mock, What Not To

- Mock only at the actual system boundary this service doesn't own: the external payment gateway, an external carrier API, another service's HTTP client. Mocking your own repository or domain code to make a unit test pass is a sign the test is testing the mock, not the code.
- Never mock the database in an integration test — that's what makes it an integration test. If a "unit" test needs a real DB to pass, it's actually an integration test; move it.
- A test that only asserts a mock was called (`Verify(x => x.DoThing())`) with no assertion on actual resulting state/behavior is a weak test — prefer asserting the observable outcome.

## Coverage & Gating

- Coverage threshold is service-defined and documented in that service's own repo (not a single global number — a read-only query service and a money-moving write service don't need identical thresholds), but every service must state one explicitly; "we didn't set a target" is not a valid threshold.
- A merge is blocked if coverage drops below the service's own documented threshold, per the platform's Quality Gate policy — not just "if tests fail."
- Flaky tests are re-run up to 3x before being flagged as flaky (non-blocking, ticketed for a fix) rather than treated as a hard failure — but a test flagged flaky twice without a follow-up fix is itself a code-review finding.

## Naming & Structure

- Test name states the scenario and expected outcome, not the method under test: `Reserve_WhenStockInsufficient_ReturnsFailure`, not `TestReserve3`.
- Arrange/Act/Assert (or Given/When/Then) sections, visually separated — a reviewer should be able to tell which section they're in without reading variable names.
- One logical assertion focus per test. A test with 10 unrelated assertions hides which one actually matters when it fails.

## What Code Review Agent checks against this file

- New domain/business logic without an accompanying unit test in the same PR.
- A "unit" test that touches a real database, broker, or wall-clock.
- Coverage regression against the service's documented threshold.
- A contract-affecting change (event schema, API shape) without a corresponding contract test update.
