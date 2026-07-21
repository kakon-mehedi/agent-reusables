---
doc_type: standard
service: null
status: accepted
layer: caching
applies_to_agents: [coding-agent, database-design-agent, code-review-agent]
---

# Caching Standards

## Strategy Selection

- **Cache-aside is the default.** App checks cache first; on miss, reads the source of truth and populates the cache with a TTL. Use this unless a specific field has a stated staleness requirement that cache-aside + TTL can't meet.
- **Write-through is the exception, not a second default.** Use it only when staleness is provably unacceptable for that specific data (e.g., an active pricing/promotion flag, where a stale read directly causes charging the wrong price) — write updates the cache and the source of truth synchronously. Reaching for write-through everywhere defeats the latency benefit caching exists for.
- Every use of write-through must state, in the design doc, *why* cache-aside's staleness window was unacceptable for that specific field — "to be safe" is not a justification.

## Invalidation

- Price-sensitive or correctness-sensitive cached data is invalidated explicitly on the relevant domain event (e.g., a price-changed event busts that SKU's cache entry) — never TTL-only. TTL-only invalidation for this class of data means there's a window, however short, where a wrong value is served with nothing wrong in the logs.
- Non-correctness-sensitive data (e.g., a product description) can rely on TTL alone.

## Key Convention

- `service:entity:id[:shard]` — e.g., `product:sku:12345`, `session:user:9876:shard3`. Consistent across services so an on-call engineer can guess a key pattern without reading that service's source.

## Hot Keys & Stampede Protection

- A small number of keys can concentrate disproportionate read traffic (a flash-sale SKU, a trending search term). Mitigate with a short-lived local in-process micro-cache in front of the distributed cache, and/or explicit key sharding (`sku:{id}:{shard}`) for the hottest known cases — don't wait for an incident to discover a hot key exists if the access pattern is predictable in advance.
- On a cache miss for a key under heavy concurrent read load, use a lock (or request-coalescing) around the recompute so N concurrent requests don't all miss and hit the source of truth simultaneously (cache stampede) — only one request recomputes; the rest wait briefly on that result.

## What Code Review Agent checks against this file

- Write-through introduced without a stated staleness-unacceptable justification.
- Correctness-sensitive cached data relying on TTL alone with no event-driven invalidation.
- A new cache key that doesn't follow the `service:entity:id[:shard]` convention.
- A predictable hot-key access pattern (e.g., a single global counter or flag read on every request) with no mitigation.
