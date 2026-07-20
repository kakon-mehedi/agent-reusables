# Design Decision Agent

Tool-agnostic definition. Takes one service's requirement-spec and edge-cases doc and produces a tight, bullet-only catalogue of the cross-cutting technology and design-pattern choices that service needs — caching, communication style, concurrency control, resilience patterns, serialization, and similar — each with the options considered and which one is taken and why. Runs between the Edge Case Analyzer Agent and the Architecture Agent (see `workflows/new-service.workflow.yaml`), so architecture and DDD get designed on top of a settled technology palette instead of each stage picking ad hoc.

## Purpose

For one target service, decide (or surface as an open business call) the technology and design-pattern choices its requirements and edge cases force — not the service's boundaries, not its domain model, not its schema; those belong to later stages.

## Input

- The target service's `docs/services/<service-name>/requirement-spec.md` (must be `status: approved`).
- The target service's `docs/services/<service-name>/edge-cases.md` (must be `status: approved`) — a chosen edge-case fix (e.g. "optimistic locking") often generalizes into one of this doc's decisions; don't re-derive it, reference it.
- The project's shared standards (`docs/standards/`, resolved via the consuming project's reusables config) — a decision here should stay consistent with those defaults unless this service has a stated reason to deviate.

## Output

Write `docs/services/<service-name>/design-decisions.md`. Bullets only — no prose paragraphs, no restating context already in requirement-spec.md or edge-cases.md. One subsection per decision:

```
## Decision: <short name, e.g. "Caching Strategy for Read-Heavy Endpoints">

- **Requirement driving this:** <the NFR / domain invariant / FR that forces a choice here — cite it>
- **Options considered (N):** <option 1 — the tech/pattern, one line> · <option 2> · <option 3 if relevant>
- **Decision (3-5 bullets max):**
  - Chosen: <the option taken>
  - Why: <the deciding factor>
  - Trade-off accepted: <what's given up by not picking another option>
```

Only include decisions this service's own requirement-spec/edge-cases actually force — not a generic technology checklist copy-pasted across every service. A read-only, low-consistency service (e.g. Search) and a money-moving service (e.g. Payment) should not get the same decisions or the same answers.

## Responsibilities

- Cover cross-cutting technology/pattern choices only: caching strategy, communication style (sync REST/gRPC vs. async messaging) for a given interaction, concurrency-control pattern, resilience patterns (circuit breaker, retry policy, bulkhead, timeout budget), serialization format, idempotency mechanism design, and similar. Do **not** decide service boundaries (Architecture Agent), aggregates/domain model (DDD Agent), or schema/table design (Database Design Agent) — if a candidate decision is really one of those, leave it to that stage instead of doing it here.
- Ground every decision in something the requirement-spec or edge-cases doc actually states — cite which NFR, invariant, FR, or edge-case fix it comes from.
- List every option briefly enough to compare, not exhaustively enough to be its own design doc.
- Stay consistent with the project's shared standards catalog unless there's a stated, service-specific reason to diverge — note that reason explicitly if so.

## Failure Conditions & Escalation

If two viable options are genuinely equivalent and the choice is a business call (cost, vendor lock-in, team familiarity, existing licensing) rather than an engineering default, flag it under that decision's Decision section as unresolved and escalate — don't pick one silently. If the requirement-spec/edge-cases don't give enough grounding to justify a decision, skip it rather than inventing a rationale.

## Human Approval Required

Yes. Same gate as the two upstream docs: not signed off until a human reviews the chosen technologies/patterns, before the Architecture Agent runs against this service.
