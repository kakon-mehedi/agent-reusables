# agent-reusables

Org-wide engineering standards and conventions that aren't tied to any single business domain — reusable across every project, not just one product.

## Contents

`docs/standards/` — engineering standards consumed by coding/architecture agents and human contributors alike:

- [`api-standards.md`](docs/standards/api-standards.md) — REST/gRPC conventions, versioning, error handling, validation
- [`git-workflow.md`](docs/standards/git-workflow.md) — branching, commit convention, merge strategy
- [`ddd-cqrs-standards.md`](docs/standards/ddd-cqrs-standards.md) — bounded contexts, aggregates, CQRS read/write model rules
- [`event-standards.md`](docs/standards/event-standards.md) — RabbitMQ/Kafka messaging conventions, event contract rules
- [`naming-conventions.md`](docs/standards/naming-conventions.md) — naming across repos, events, routing keys, code, env vars
- [`folder-structure.md`](docs/standards/folder-structure.md) — Clean Architecture + Vertical Slice folder layout

## Usage

Clone this repo alongside each consuming project (e.g. as a sibling directory) and reference these docs from project-specific standards. Project-specific overlays (concrete naming prefixes, exchange names, bounded contexts, etc.) belong in the consuming project's own docs, not here.
