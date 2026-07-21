---
doc_type: standard
service: null
status: accepted
layer: deployment
applies_to_agents: [scaffold-agent, coding-agent, docker-agent, deployment-agent, code-review-agent]
---

# Deployment Standards (Docker & Kubernetes)

## Docker

- Multi-stage builds, mandatory: an SDK/build-tools stage compiles and publishes; the final stage copies only the published output onto a minimal runtime base image. The build toolchain never ships to production — smaller image, smaller attack surface.
- Dependency-restore layer is cached separately from the source-copy layer (`COPY *.csproj` + restore, *then* `COPY . .` + build) — so a source-only change doesn't invalidate the dependency cache on every build.
- No secret is ever baked into an image layer, including one that's later removed in a subsequent layer — layers are still present in the image history. Secrets are injected at runtime only (see [`security-standards.md`](security-standards.md)).
- Base images are scanned for known CVEs as part of the build; a critical/high finding blocks the build rather than shipping and fixing later.

## Kubernetes

- Autoscaling (HPA) uses the metric that actually reflects load for that workload — CPU for stateless request-handling pods, a custom metric (e.g., queue depth) for consumer/worker pods where CPU alone lags the real signal.
- Every deployment declares resource requests *and* limits — no pod runs with unbounded resource usage, which is what lets a single noisy pod starve its neighbors on a shared node.
- Liveness and readiness probes are mandatory and distinct: readiness gates whether the pod receives traffic (e.g., "warmed up, DB connection established"); liveness gates whether the pod gets restarted (e.g., "deadlocked, kill it"). Conflating them causes either premature traffic routing or unnecessary restart loops.
- Non-secret configuration (feature flags, topology manifests) lives in a `ConfigMap`; credentials and keys live in a `Secret` (or an external secret manager reference) — never mixed in the same object, so a config diff review never accidentally exposes a credential.
- Rolling updates by default; a service opts into a stricter strategy (canary, blue-green) only when its criticality/blast-radius justifies the added deployment complexity — not applied uniformly regardless of risk.

## What Code Review Agent / Docker Agent check against this file

- A single-stage Dockerfile, or a final image that still contains SDK/build tooling.
- A secret referenced via a build ARG or `COPY`'d config file instead of runtime injection.
- A deployment manifest with no resource limits, or no readiness/liveness probes.
- Autoscaling configured on CPU alone for a queue-consumer workload.
