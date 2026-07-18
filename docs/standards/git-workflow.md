---
doc_type: standard
service: null
status: accepted
layer: coding
applies_to_agents: [coding-agent, scaffold-agent, code-review-agent, ticket-agent]
---

# Git & GitHub Flow Standards

- **Flow**: GitHub Flow — `main` is always deployable, no direct commits to `main`, every change via PR.
- **Branch naming**: `<type>/<ticket-id>-<short-desc>`, e.g. `feat/PROJ-142-add-search-filter`, `fix/PROJ-88-double-submit`.
- **Commit convention**: Conventional Commits — `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`. Body explains *why*, not *what* (the diff already shows what).
- **Merge strategy**: squash-merge to `main` by default; PR title becomes the squash commit message.
- **PR size**: one vertical slice per PR. If a PR touches more than one Application/Features folder, split it.
- Never `--force` push to `main`, never `--no-verify`, never skip CI to merge faster.
