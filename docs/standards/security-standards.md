---
doc_type: standard
service: null
status: accepted
layer: security
applies_to_agents: [coding-agent, code-review-agent, security-review-agent]
---

# Security Standards

This file states the generic, project-agnostic security patterns. A consuming project's concrete identity providers, role catalog, and specific compliance scope (e.g. this platform's SSO/RBAC decisions in `kart-requirements.md` §24) layer on top of this, the same way `kart-conventions.md` layers Kart-specific naming on top of `naming-conventions.md`.

## AuthN

- User-facing clients: OAuth2 Authorization Code flow (with PKCE for public/mobile clients).
- Service-to-service: OAuth2 Client Credentials flow — a service never uses a user's token to call another service; it uses its own service-principal credentials.
- SSO federation (SAML/OIDC), if a project needs it, terminates at a single identity-issuing service — no other service ever validates an external IdP's assertion directly or holds an external IdP client secret. This keeps the IdP integration surface (and its blast radius if compromised) in one place.

## AuthZ

- Claim/scope-based, not per-service role tables. One issuer mints the claim; every other service is a consumer of it, never a second source of truth for "who can do what" — a locally-defined permission table per service silently drifts from every other service's definition within a release cycle or two.
- Checked twice: coarse-grained at the API Gateway (reject before the request reaches a service), fine-grained again at the owning service for anything the gateway can't evaluate (e.g., a monetary cap that depends on the specific resource being acted on).
- Denies by default. A missing or unrecognized scope/role is a 403, never an implicit allow.

## Secrets & Configuration

- No secret (API key, DB credential, signing key) is ever committed, hardcoded, or baked into a Docker image layer. Secrets come from the runtime environment (K8s `Secret` + an external secret manager for anything beyond dev), never from a config file checked into git.
- Secrets and non-secret config are never in the same object — non-secret config (feature flags, topology JSON) goes in a `ConfigMap`-equivalent; only actual secrets go in `Secret`-equivalent storage.
- Rotate long-lived credentials on a documented schedule; a credential with no rotation plan is treated as a finding, not an oversight to fix "later."

## Encryption

- TLS everywhere, including internal service-to-service traffic — "it's inside the cluster" is not a reason to skip it.
- PII columns encrypted at rest (e.g. AES-256); payment/card data is never stored at all — tokenized by the gateway provider, so this codebase never becomes a target for card data regardless of how well it's otherwise secured.
- Token lifecycle: short-lived access tokens, rotating refresh tokens, and an explicit revocation path (e.g. a revocation list) for logout/compromise — a token that can't be revoked before its natural expiry is a standing risk.

## Input & API Security

- Validate at the API boundary (see [`api-standards.md`](api-standards.md)'s Validation section) *and* enforce invariants in the domain layer — the API-layer check is a fast-fail convenience, never the only line of defense.
- Rate limiting, tiered by caller trust level (anonymous < authenticated < partner/service).
- Mandatory `Idempotency-Key` on money-moving or otherwise non-retry-safe writes (already required by [`api-standards.md`](api-standards.md); called out again here because it's also a security control against replay-driven double-processing, not purely a correctness one).

## What Security Review Agent checks against this file

- Hardcoded secrets or credentials anywhere in the diff (including test fixtures — a "fake" key that looks real enough to be mistaken for one is still a finding).
- A new endpoint with no AuthZ check, or an AuthZ check that fails open instead of closed.
- PII stored unencrypted, or logged (see [`observability-standards.md`](observability-standards.md)).
- A new outbound integration (payment, IdP, external API) that doesn't route through a single owned adapter — direct, scattered calls to an external provider from multiple places is both a security and a decoupling problem (see [`coding-standards.md`](coding-standards.md)'s Adapter guidance).
