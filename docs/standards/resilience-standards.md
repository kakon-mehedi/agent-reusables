---
doc_type: standard
service: null
status: accepted
layer: resilience
applies_to_agents: [coding-agent, code-review-agent, architecture-agent]
---

# Resilience Standards

Every outbound call — synchronous HTTP/gRPC or an async consumer's downstream call — is a potential failure point. These patterns are what turns "a dependency is slow or down" into graceful degradation instead of a cascading failure across the whole platform. (Messaging-specific retry/DLQ policy for published/consumed events is [`event-standards.md`](event-standards.md)'s scope; this file covers synchronous outbound calls and general fault-isolation patterns.)

## Circuit Breaker

- Every synchronous outbound call (to another service, or to an external system) is wrapped in a circuit breaker — closed (normal) → open (failing fast, not calling the dependency) → half-open (probing recovery) → closed.
- When open, fail fast with a clear error or a defined fallback — never let callers pile up waiting on a dependency that's already known to be down.
- Breaker thresholds (failure rate, sample window, open duration) are documented per dependency, not left at a library default nobody reviewed.

## Retry with Backoff

- Exponential backoff with jitter, bounded to a small number of attempts (2-3 for a synchronous call a user is waiting on; more is a UX problem, not a resilience win).
- Only retry idempotent or explicitly idempotency-keyed operations — retrying a non-idempotent write is a correctness bug, not a resilience feature.
- A retry that will predictably fail again immediately (e.g., the circuit breaker is already open) should not fire — check breaker state before retrying, don't let retry logic fight breaker logic.

## Bulkhead Isolation

- Separate connection pools / thread pools / semaphores per dependency, so one slow or saturated dependency can't starve requests to a healthy one. A single shared pool for "all outbound HTTP calls" defeats the purpose of having a circuit breaker per dependency.
- Size each bulkhead to the dependency's actual capacity and this service's actual concurrency needs — an oversized shared pool just delays the point where saturation becomes visible.

## Timeout Budgets

- Every outbound call has an explicit timeout — no call waits indefinitely on a hung dependency.
- Timeout budgets are allocated top-down from the end-to-end latency budget (see the consuming project's NFR targets) — a call three hops deep can't have the same timeout as the whole request's budget, or a single slow leaf call blows the entire budget with nothing left for the caller to even format an error response.

## Graceful Degradation vs. Cascading Failure

- Define, per critical dependency, what "degraded but still useful" looks like — e.g., serve a cached/stale price rather than fail the page entirely, if staleness is acceptable for that specific read; fail the write entirely if it's a money-moving operation where a stale or guessed answer is worse than an explicit error.
- A dependency failure must never propagate as an unhandled exception that takes down an unrelated code path in the same process — isolate the failure to the feature that actually depends on the failing thing.

## What Code Review Agent checks against this file

- A new synchronous outbound call with no circuit breaker, no timeout, or an unbounded/no-jitter retry loop.
- A retry applied to a non-idempotent operation.
- Multiple unrelated outbound dependencies sharing one connection/thread pool.
- A dependency failure with no defined fallback or degraded behavior — just an unhandled exception bubbling up.
