---
doc_type: standard
service: null
status: accepted
layer: documentation
applies_to_agents: [documentation-agent, coding-agent, code-review-agent]
---

# Documentation Standards

## Docs-as-Code

- Documentation lives in Markdown, in the same repo as the thing it describes, versioned alongside the code — never in an external wiki that drifts silently out of sync.
- A PR that changes observable behavior (an API shape, an event contract, a config option) updates its documentation in the *same* PR — "docs updated" is a merge gate, not a follow-up task. A PR with no doc diff for a behavior change should be treated as incomplete, not just under-documented.
- Conservative default: when unsure whether a change is "significant enough" to document, document it — a human reviewer can trim, but an agent that under-documents by default lets real gaps accumulate silently.

## Service README (per service repo)

Every service repo's own `README.md` is operational, not a design record (the design record — requirements, architecture, DDD model — lives centrally, e.g. this platform's `docs/services/<name>/`). It states, at minimum:
- What the service does, in one paragraph a new engineer can act on immediately.
- How to run it locally (setup, dependencies, environment variables it needs — not their values).
- How to run its tests.
- Where its design record and API/event contracts live (link out, don't restate).
- Who/what owns it operationally (on-call rotation or equivalent, if applicable).

## ADR Convention

- One architectural decision per file. A file that bundles several unrelated decisions makes it impossible to supersede one without disturbing the others.
- Immutable once accepted — a changed mind is a **new** ADR that supersedes the old one (old ADR's status becomes `superseded`, with a link forward), never an edit to the original. This preserves the actual history of what was decided when and why, which is the entire point of an ADR trail.
- `status` field is mandatory and one of: `proposed`, `accepted`, `rejected`, `superseded`. A status-less ADR can't be trusted by a later reader or an agent doing retrieval.
- Use [`adr-template.md`](adr-template.md) as the starting structure (Status / Context / Decision / Consequences) — don't invent a per-project ADR shape.

## What Documentation Agent / Code Review Agent check against this file

- A merged PR that changed an API/event contract or config surface with no corresponding doc update.
- A service repo missing the minimum README sections above.
- An ADR that edits a previously-accepted decision in place instead of superseding it with a new file.
- A new ADR with no `status` field, or a `status` inconsistent with its own Decision section (e.g., body reads as still under discussion but status says `accepted`).
