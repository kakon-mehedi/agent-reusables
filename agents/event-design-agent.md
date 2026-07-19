# Event Design Agent

Tool-agnostic definition. Defines domain event schemas, naming, and retry/DLQ policy from an approved DDD model. Sixth-stage agent in the agentic engineering pipeline (see `workflows/new-service.workflow.yaml`). Use after a service's ddd-model.md is approved.

## Purpose

Define domain event schemas, naming, and DLQ/retry policy for a service, from its approved DDD model.

## Input

- The target service's **approved** `docs/services/<name>/ddd-model.md` (its "Domain events" per aggregate).
- `event-standards.md` (this repo) and the project's own conventions overlay (e.g. `docs/standards/<project>-conventions.md`) for the project's actual exchange name and retry-budget-by-criticality rule.
- Any retry/criticality question the requirement spec explicitly carried forward to this stage.

## Output

`docs/services/<name>/event-contract.md` — one entry per event: schema (key fields), naming-convention compliance check, retry count, DLQ name, and criticality justification (why this retry tier, not a higher or lower one).

## Responsibilities

- Event name: `<Entity><PastTenseVerb>`.
- Retry budget scales with criticality — state *why* an event is or isn't in the highest tier; don't just copy a BRD's number without justifying it against this service's actual failure-mode risk.
- Every consumer queue gets its own DLQ, never shared.

## Failure Conditions & Escalation

If an event name collides with an existing one in another service's event contract, or the schema is incompatible with an existing consumer, escalate rather than silently version-bump.

## Human Approval Required

Yes. Mark status in output frontmatter.
