---
doc_type: standard
service: null
status: accepted
layer: database
applies_to_agents: [database-design-agent, coding-agent, code-review-agent]
---

# Database Standards

The write-store-is-truth / read-store-is-rebuildable *principle* is [`ddd-cqrs-standards.md`](ddd-cqrs-standards.md)'s scope. This file covers the concrete mechanics of implementing that principle in a relational write store and a document read store.

## PostgreSQL (Write Store)

- Every table's indexes are chosen for a documented, actual query pattern — "might need it later" is not a reason to add an index (indexes cost write throughput and storage, so an undocumented one is a liability, not free insurance).
- High-volume, time-series-shaped tables (append-heavy, queried mostly by recent range) use range partitioning by date — bounds index size and makes archival of old partitions cheap, instead of one ever-growing table.
- Contention-heavy invariants (e.g., a quantity that must never go negative under concurrent writers) use row-level locking (`SELECT ... FOR UPDATE`) scoped as narrowly as possible — lock the specific row(s) the invariant depends on, never a broader range/table lock as a shortcut.
- Schema changes are always a versioned, reviewed migration — never a manual `ALTER TABLE` run by hand against an environment. A migration is forward-only in production; a mistake is fixed by a new migration, not by editing history.
- Uniqueness constraints (e.g., an idempotency key) are enforced at the database level, not only checked in application code — a race between two application-level checks can both pass; a DB unique constraint can't.

## MongoDB (Read Store)

- Collections are denormalized on purpose — embed the fields a query actually needs (e.g., category name inside a product document) to avoid a join MongoDB can't do across shards. Denormalization trades storage and eventual staleness for read latency; that trade must be a stated decision, not an accident of "however the data ended up shaped."
- Shard key selection is driven by two things: even write distribution (avoid a hot shard) and query locality (the shard key should match the field most queries filter/browse by) — document which one drove the choice when they'd conflict.
- Replica sets use majority write concern for anything serving as a read model of record — an unacknowledged or single-node write is a durability gap for data that's supposed to be rebuildable-but-authoritative-until-rebuilt.
- A read model must always be rebuildable from the write store + event log alone. If rebuilding it requires any data that only ever existed in the read model, that's a CQRS violation — some piece of truth leaked into the "derived" side.

## Migrations & Schema Evolution

- Every schema change (relational migration, or a read-model's document shape change) is documented with its rationale at the time it's made — index reasoning, partitioning key reasoning, shard key reasoning — so a later engineer isn't reverse-engineering *why* from the DDL alone.
- A read-model shape change is a projection-consumer change, not a write-store migration — it's deployed by changing the consumer and replaying/rebuilding the projection, never by hand-editing documents in place.

## What Code Review Agent / Database Design Agent check against this file

- A new index with no stated query pattern justifying it.
- A contention-prone write path with no explicit locking strategy.
- A schema change applied outside the migration mechanism.
- A denormalized read-model field with no traceable source in the write model or event log.
