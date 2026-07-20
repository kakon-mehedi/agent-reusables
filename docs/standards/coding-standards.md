---
doc_type: standard
service: null
status: accepted
layer: coding
applies_to_agents: [scaffold-agent, coding-agent, code-review-agent]
---

# Coding Standards — Clean Code, SOLID, Design Patterns, Decoupling

This is the standards-doc referenced by the platform blueprint's Quality Gate table ("Coding Standards | PR review | Code Review Agent + linter | Zero linter errors, **standards doc compliance**") — before this file existed, that gate had nothing concrete to check compliance against. It complements, not duplicates: [`folder-structure.md`](folder-structure.md) owns physical layout, [`ddd-cqrs-standards.md`](ddd-cqrs-standards.md) owns domain/read-write-model rules, [`api-standards.md`](api-standards.md) owns HTTP/error-handling shape. This file owns the code *inside* a vertical slice — how a class/function/module is written, not where it lives or what it exposes.

## SOLID (the non-negotiable baseline)

- **Single Responsibility** — a class has one reason to change. A `Handler` that also formats emails or computes tax has two callers who can each break it for unrelated reasons; extract the second concern.
- **Open/Closed** — new behavior is added by adding a new implementation of an existing interface (a new `IDiscountStrategy`), not by adding another `if (type == "X")` branch to an existing one. A growing `switch`/`if-else` chain on a type/enum is the concrete signal this principle is being violated.
- **Liskov Substitution** — any implementation of an interface must be usable wherever the interface is expected, with no caller needing to know which concrete type it got. A subtype that throws `NotSupportedException` on part of its parent's contract is the smell here.
- **Interface Segregation** — prefer several small, purpose-specific interfaces (`IReserveStock`, `IReleaseStock`) over one fat interface (`IInventoryEverything`) that forces every implementer to stub methods it doesn't need.
- **Dependency Inversion** — `Application`/`Domain` depend on interfaces they own; `Infrastructure` implements those interfaces. Never the reverse. This is the same rule [`folder-structure.md`](folder-structure.md) states as "`Domain` never references `Infrastructure`" — DIP is *why* that rule exists, not a separate one.

## Decoupling

- All cross-layer dependencies are injected (constructor injection), never resolved via a static accessor, service locator, or `new`-ed up directly inside a class that isn't a composition root. A class that can't be unit-tested without spinning up a real database or broker has a decoupling defect, not just a testing inconvenience.
- A vertical slice's `Handler` depends on interfaces (`IOrderRepository`, `IEventPublisher`), never on another slice's `Handler` directly. Cross-slice reuse goes through `Domain` (shared aggregates/services) or a shared `Application`-level abstraction — never by one Feature folder importing another Feature folder's internals.
- External systems (payment gateway, carrier API, an external IdP) are wrapped behind an interface owned by this codebase (an Adapter/Anti-Corruption Layer), so a vendor swap or SDK upgrade touches one Infrastructure implementation, not every call site.
- No circular references between projects/assemblies. If `Domain` needs something `Infrastructure` provides, that's a missed abstraction — `Domain` defines the interface, `Infrastructure` implements it.

## Design Patterns — reach for these, not a bigger `if`

Patterns are a response to a recurring shape of problem, not a checklist to apply everywhere. Use the table below to recognize the shape; don't introduce a pattern's ceremony (interface + multiple implementations + a factory) for a choice that will only ever have one implementation.

| Shape of the problem | Pattern | Where it already applies in Kart |
|---|---|---|
| Behavior varies by a type/enum and that set of types will grow | Strategy | Pricing/discount calculation per promotion type; carrier-selection logic per shipping method |
| Object construction has real conditional complexity (not just a constructor call) | Factory / Factory Method | Building a payment-gateway client per configured provider |
| Need to add behavior (logging, caching, retry) around a call without changing its callers | Decorator | Wrapping a repository or gateway client with a caching or retry layer |
| A read model/query has many optional filter combinations | Specification | Search/faceted-filter query construction |
| One aggregate's state change needs to notify other in-process code without a direct reference | Observer / domain events (in-process) | Domain events raised by an aggregate, dispatched after the transaction commits (in-process — distinct from the platform's cross-service RabbitMQ/Kafka events) |
| A command/query needs validation → handler → post-processing without the controller knowing all three | Mediator | The `Command`/`Query` → `Handler` shape [`folder-structure.md`](folder-structure.md) already mandates per vertical slice *is* this pattern — implement it with a mediator library (e.g. MediatR for .NET), don't hand-rolled-dispatch it differently per slice |
| Persistence needs to be swappable/testable independent of EF/Mongo specifics | Repository | Already implied by CQRS's write/read model split — one repository abstraction per aggregate root, never a generic `IRepository<T>` that leaks ORM-specific query capabilities |
| Two incompatible interfaces need to talk (an external SDK's shape vs. this codebase's own interface) | Adapter | External carrier API, external IdP token exchange (see `kart-platform`'s ADR-0002-era SSO decisions) |

**Anti-pattern check, every review:** if a "pattern" was introduced for a single implementation with no near-term second one, that's premature abstraction — flag it the same as a missing pattern. Three concrete, similar call sites justify extracting an abstraction; two might, one never does.

## Clean Code — function and class hygiene

- A function does one thing at one level of abstraction. If you need "and" to describe what it does, split it.
- Prefer early returns/guard clauses over nested `if`. More than 2 levels of nesting in a method is a refactor signal.
- No magic numbers/strings — name the constant, even if it's only used once, when the value's meaning isn't obvious from context.
- Parameter lists longer than ~4 are a signal the parameters belong in their own type (a request/command object), not more positional arguments.
- Comments explain **why**, never **what** — if a comment restates the code, delete the comment or rename the thing it's describing instead. This mirrors the same rule this platform's own agents are held to (see `AGENTS.md`-style guidance in any consuming project).
- Boolean parameters that change a function's behavior (`Process(order, true)`) are a code smell — prefer two named methods or an enum parameter; a reader at the call site should never have to jump to the signature to know what `true` means.
- Primitive obsession: a `string` used everywhere a `Sku`, `Money`, or `EmailAddress` value object would prevent an entire class of mix-up bugs (e.g., passing a `userId` where an `orderId` was expected) — DDD value objects (per [`ddd-cqrs-standards.md`](ddd-cqrs-standards.md)) are Clean Code's answer to this, not a separate concern.

## DRY vs. premature abstraction

- Duplication across **two** call sites is not yet a violation worth fixing — wait for a third genuinely identical case before extracting a shared abstraction. An abstraction built from two data points is usually wrong in a way that's more expensive to unwind than the duplication was.
- Don't build a generic/configurable solution for a hypothetical fourth case that doesn't exist yet in a ticket. Extend when the fourth case actually arrives.

## What Code Review Agent checks against this file

- SOLID violations described above, with the specific principle named in the review comment (not just "this feels off").
- Missing dependency injection / hidden `new`-ing of infrastructure concerns inside `Domain` or `Application`.
- A pattern applied where a simpler direct implementation would do (over-engineering), and the reverse — a growing type-switch that should have become Strategy.
- Nesting depth, parameter-list length, and function length outliers relative to the rest of the codebase (not a hard universal number — flag outliers, not every function over N lines).
- Any comment that restates the adjacent code instead of explaining a non-obvious why.
