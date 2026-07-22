---
doc_type: standard
service: null
status: accepted
layer: process
applies_to_agents: [all]
---

# Agent Efficiency Standards — incremental work, caching, and scan discipline

Project-agnostic rules for not re-spending tokens on work that is already done or not needed. Every stage agent, review agent, and orchestrator in this pipeline follows these by default. A consuming project's own `AGENTS.md`-equivalent may reference this file and layer project-specific pointers on top (e.g. "check `README.md`'s status board first") — it should not restate these rules.

## Always

- **Check status before reprocessing.** Read a doc's frontmatter `status` field first. If it is `approved`/`accepted` and none of its declared upstream inputs have changed since, treat it as done — do not re-read it in full, re-derive its content, or re-verify it "just in case." A completed, still-valid artifact is a cache hit: (inputs unchanged) + (output exists) + (status approved) → skip.
- **Read only what your stage's contract requires.** Each stage agent's own definition names its Input. Read that, and only that, plus whatever a specific task genuinely calls for — not every sibling doc "in case it's relevant."
- **Read a file once per task, not once per reference to it.** If you already have a file's content in context from earlier in the same task, reuse it — don't re-open it to check something you already read, unless you have a concrete reason to believe it changed since (e.g. another agent edited it mid-task).
- **When resuming interrupted multi-step work** (a paused pipeline, an orchestrated multi-agent run), resume from the last completed, still-valid step. Do not restart a chain from the beginning "to be safe" — verify the last checkpoint is genuinely valid, then continue past it.
- **When fixing or sweeping many already-processed items for one narrow change** (e.g. flipping a status field, correcting one citation), make the minimal targeted edit. Do not regenerate the whole document's content to make one small correction.
- **Use the project's own reading-order/index doc** (its `AGENTS.md`, `README.md` status board, or equivalent) to find the one file that answers a question, instead of grepping or scanning the whole repository tree.

## Never

- Never re-open or re-summarize a file you already hold in context without a concrete reason to think it changed.
- Never re-derive information (a status, a decision, a dependency graph) by rescanning many files when a single index/status doc already states it.
- Never treat "double-checking everything from scratch" as safer by default — it is not free, and a targeted verification of what actually changed is both cheaper and more precise than a full rescan.
- Never regenerate an entire artifact to make a one-line fix.

## Session scope and lifecycle

Every subagent spawned is a full additional set of model requests, and every uncompacted turn re-sends the session's entire history. These compound: a session that fans out to many agents *and* stays open for hours *and* never compacts pays for all three at once.

- Bound a session to one unit of work (one service, one pipeline stage, one review, one fix). Finish it, say so plainly, and stop — don't drift into adjacent follow-on work the human didn't ask for in this pass.
- Compact between distinct tasks in the same session; clear when switching to a genuinely different task or service. Don't carry one service's full design-doc context into work on an unrelated service — it adds re-read cost with no reuse benefit.
- Don't let context grow unbounded on the assumption that "the session isn't done yet." If a session's own history is now a meaningful fraction of what gets re-sent each turn, that is the signal to compact — waiting until the session "feels" too long means paying the cost for longer than necessary.
- When resuming a session after a long gap (waiting on human approval, a scheduled resume), compact first rather than resuming with the full pre-gap context intact. Re-check only what changed since the pause (status fields, new commits) — don't reload everything "to be safe"; that's the same waste the "resume from last checkpoint" rule above already forbids.

## Fan-out scale — bound orchestration to what the task needs

- Spawn only the stage agent(s) the task actually names. "Do X for `<service>`" means running the stage(s) X names — not preemptively running the rest of the pipeline "since we're already here." Each pipeline stage requires an approved upstream input anyway (see `Always` above); running ahead of that is wasted work, not a head start.
- When a task spans many items (e.g. a sweep across many services), state the batch size up front and keep it small (a handful, not the full backlog), compacting or checkpointing between batches. A vague "review all of them" invites the largest possible fan-out; a stated batch plan bounds it before any tokens are spent.
- Reach for multi-agent orchestration (parallel fan-out, workflow-style tooling) only when the task is genuinely wide — many independent items, or a review that benefits from independent adversarial passes. It is not the default posture for single-service or single-stage work: one stage on one service is one agent call, not an orchestrated run through every specialist.
- Default to one targeted verification pass per artifact. A second or third re-verify pass on the same artifact needs a concrete reason (something changed since the last pass, a specific concern raised) — "double-checking again to be safe" is the same false economy called out above, just applied to verification instead of reading.
- Scope subagent prompts narrowly: state what to check and what to explicitly skip (e.g. "check API contract consistency only, not naming or formatting"). An open-ended "review everything" prompt invites the agent to re-read and re-verify more than the task needs, and that cost is paid in full even when the extra scope finds nothing.

## Consider (explicit project decision, not a default)

- Running orchestration-heavy or high-fan-out stages on a cheaper model tier than the main session, when a project's tooling supports per-agent model overrides. This repo's own `registry.yaml` currently sets `model_tier: inherits` for every stage agent as a deliberate choice (no multi-provider routing, see the comment there) — changing that is a real quality/cost tradeoff, not a formatting change, and should be a decision the human makes explicitly per project rather than something silently defaulted here.
