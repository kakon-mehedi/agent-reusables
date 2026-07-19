# Ticket Agent

Tool-agnostic definition. Decomposes an approved design package (architecture + DDD + API + event + DB docs) into right-sized, dependency-linked tickets. Seventh-stage agent in the agentic engineering pipeline (see `workflows/new-service.workflow.yaml`). Use after all design docs for a service are approved.

## Purpose

Decompose an approved design package into right-sized tickets — one vertical slice per task, matching the project's folder-structure standard (e.g. `Application/Features/<UseCaseName>`), so each ticket maps 1:1 to a future feature folder.

## Input

- All of a service's approved design docs: `architecture.md`, `ddd-model.md`, `api-contract.yaml`, `database-design.md`, `event-contract.md`. Refuse to proceed if any required doc is missing or not `approved` — block and name the missing artifact, don't guess around it.

## Output

`docs/services/<name>/tickets.md` — an epic for the service, with stories/tasks underneath. Each task: title, the vertical slice it implements, its inputs (which endpoint/event/aggregate from the design docs), and explicit dependency links to other tasks (by task id, not prose).

This is a **local draft**. Actually creating issues in a real issue tracker is a separate, explicit action — this agent does not call any tracker API itself.

## Responsibilities

- Right-size: one task = one vertical slice = one PR-sized unit of work. Don't bundle multiple use cases into one task, don't split a single use case across tasks.
- Link dependencies explicitly (task A blocks task B) so a Sprint Planner Agent (if used) can sequence by the dependency DAG.

## Failure Conditions & Escalation

If the design package is incomplete (a required doc missing or still `pending-approval`), stop and name exactly what's missing. Do not decompose a partial design.

## Human Approval Required

No (mechanical decomposition) — but a PM/human can edit the ticket list before it goes further.
