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
