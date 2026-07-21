---
doc_type: standard
service: null
status: accepted
layer: observability
applies_to_agents: [coding-agent, code-review-agent, monitoring-agent]
---

# Observability Standards

Three pillars, all mandatory per service, none optional "add later":

## Structured Logging

- JSON only, never free-text strings. A log line is a machine-parseable record, not prose for a human tailing a terminal.
- Mandatory fields on every log line: `traceId`, `service`, `level`, and whichever entity id is relevant to the operation (`orderId`, `userId`, etc.).
- Log level discipline: `Error` = the operation failed and needs attention; `Warn` = degraded but recovered/recoverable; `Info` = a significant business event (order created, payment failed), not every method entry/exit; `Debug` = disabled in production by default.
- Never log secrets, tokens, full card numbers, or raw PII — log the entity id, not the payload that identifies a person.

## Metrics (RED method)

- Every service exposes **R**ate, **E**rrors, **D**uration for each of its own endpoints/consumers — this is the baseline, not a nice-to-have per service.
- Business metrics (orders/min, cart abandonment rate, redemption rate) are named and owned by the service that produces the underlying event, not bolted on generically by Analytics.
- A metric without an alert threshold is incomplete — defining a metric and never deciding when it's "bad" just produces a dashboard nobody looks at until an incident.

## Distributed Tracing

- W3C Trace Context propagated through both HTTP headers and message headers — a trace must survive crossing from a synchronous call into an async event and back, or the whole point of tracing an order across 8+ services is lost.
- Every service boundary crossing (HTTP call out, message publish, message consume) creates a new span; a span name states the operation, not just the service (`ReserveInventory`, not `InventoryService`).
- 100% trace sampling on the critical/money-moving path; sampling can be reduced on read-heavy, low-criticality paths (search, catalog browse) once volume makes 100% uneconomical — this is a per-service, documented trade-off, not a silent default.

## What Code Review Agent checks against this file

- A new endpoint/consumer with no structured log on both the success and failure path.
- A log statement that free-text-interpolates a value instead of passing it as a structured field.
- A new outbound call (HTTP or message publish) that doesn't propagate the incoming trace context.
- PII or secrets appearing in a log statement.
