# agent-reusables

Org-wide engineering standards and conventions that aren't tied to any single business domain — reusable across every project, not just one product.

## Contents

`docs/standards/` — engineering standards consumed by coding/architecture agents and human contributors alike:

- [`api-standards.md`](docs/standards/api-standards.md) — REST/gRPC conventions, versioning, error handling, validation
- [`coding-standards.md`](docs/standards/coding-standards.md) — SOLID, decoupling rules, a design-pattern reach-for-this table, Clean Code function/class hygiene, DRY-vs-premature-abstraction judgment
- [`git-workflow.md`](docs/standards/git-workflow.md) — branching, commit convention, merge strategy
- [`ddd-cqrs-standards.md`](docs/standards/ddd-cqrs-standards.md) — bounded contexts, aggregates, CQRS read/write model rules
- [`event-standards.md`](docs/standards/event-standards.md) — RabbitMQ/Kafka messaging conventions, event contract rules
- [`naming-conventions.md`](docs/standards/naming-conventions.md) — naming across repos, events, routing keys, code, env vars
- [`folder-structure.md`](docs/standards/folder-structure.md) — Clean Architecture + Vertical Slice folder layout
- [`testing-standards.md`](docs/standards/testing-standards.md) — test pyramid, mocking policy, coverage gating, flaky-test policy
- [`observability-standards.md`](docs/standards/observability-standards.md) — structured logging, RED metrics, distributed tracing
- [`security-standards.md`](docs/standards/security-standards.md) — AuthN/AuthZ patterns, secrets management, encryption, input security
- [`resilience-standards.md`](docs/standards/resilience-standards.md) — circuit breaker, retry/backoff, bulkhead isolation, timeout budgets
- [`caching-standards.md`](docs/standards/caching-standards.md) — cache-aside vs write-through, key convention, invalidation, hot-key/stampede mitigation
- [`database-standards.md`](docs/standards/database-standards.md) — PostgreSQL write-store and MongoDB read-store mechanics
- [`deployment-standards.md`](docs/standards/deployment-standards.md) — Docker multi-stage builds, Kubernetes autoscaling/config/secrets
- [`documentation-standards.md`](docs/standards/documentation-standards.md) — docs-as-code, service README contents, ADR convention rules
- [`adr-template.md`](docs/standards/adr-template.md) — blank Architecture Decision Record template

`agents/` — a tool-agnostic agentic engineering pipeline (BRD → requirement spec → architecture → DDD model → API/database/event design → tickets), one markdown definition per stage, each with its own approval-gate policy:

- [`requirement-agent.md`](agents/requirement-agent.md), [`architecture-agent.md`](agents/architecture-agent.md), [`ddd-agent.md`](agents/ddd-agent.md), [`api-design-agent.md`](agents/api-design-agent.md), [`database-design-agent.md`](agents/database-design-agent.md), [`event-design-agent.md`](agents/event-design-agent.md), [`ticket-agent.md`](agents/ticket-agent.md)

`workflows/` — the DAG that sequences those agents:

- [`new-service.workflow.yaml`](workflows/new-service.workflow.yaml) — stage order, dependencies, and which stages require human approval. Data only, not an execution engine — a human or coding agent reads it to decide what runs next.

## Usage

Clone this repo alongside each consuming project (e.g. as a sibling directory) and reference these docs from project-specific standards. Project-specific overlays (concrete naming prefixes, exchange names, bounded contexts, etc.) belong in the consuming project's own docs, not here.
