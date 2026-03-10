# Caching

## Generation Instructions

Caching provides a shared cache client used by other capabilities for read-through caching, session storage (when Auth uses server-side sessions), ephemeral state (Presence active-user tracking), and rate limiting counters. Generating a unified cache client ensures all capabilities share one connection pool and one key namespace convention.

Generate a cache client module at `lib/cache` that wraps the provider client and exposes: `get(key)`, `set(key, value, ttlSeconds?)`, `del(key)`, `exists(key)`, `invalidate(pattern)`. The wrapper normalizes provider differences so callers are not coupled to provider-specific APIs.

Key namespacing: prefix every key with a capability identifier and a separator to prevent collisions. The convention is `{capability}:{key}`. For example, Presence stores `presence:document:123`, Feature Flags stores `flags:user:456`. Generate a `cacheKey(capability, ...parts)` helper that enforces this convention. Capabilities must use this helper — raw string keys are not acceptable.

Resolve the provider from `config.provider`:

- `redis`: Generate a Redis client using `ioredis`. Connection URL from `REDIS_URL`. Generate connection pooling with a reasonable default pool size. If Background Jobs also uses Redis, share the connection instance — do not create two separate Redis connections.
- `memcached`: Generate a Memcached client using `memjs` or equivalent. Connection string from `MEMCACHED_SERVERS`. Memcached does not support key pattern invalidation — generate a warning comment wherever `invalidate(pattern)` is called noting this limitation.
- `in-memory`: Generate an LRU cache using `lru-cache` or equivalent. No external infrastructure. Safe for single-process, single-instance deployments only. Generate a startup warning: `"Cache provider is in-memory. This is not safe for multi-instance deployments — cache state is not shared between instances."` If `infrastructure.scalability.horizontal` is `true`, escalate to a startup error and refuse to start.

TTL defaults: each capability that uses the cache declares its own TTL appropriate to the data's freshness requirements. The cache client accepts TTL per `set` call. Generate a `DEFAULT_TTL_SECONDS` constant (300 — five minutes) used when a caller does not specify a TTL, but capabilities should specify explicitly.

Cache invalidation: generate event-based invalidation hooks where capabilities emit events that should trigger cache invalidation. For example, when a Feature Flag is updated, the flags cache for all users is invalidated. Wire these in the capability's event handler, not in the cache module itself — the cache module is passive.

## Configuration

```yaml
capability: caching
type: infrastructure
version: 1.0

exposes:
  - cache-client: >
      lib/cache — get, set, del, exists, invalidate operations.
      cacheKey(capability, ...parts) helper for namespaced keys.
      Imported by any capability that needs caching or ephemeral state storage.

config:
  provider: redis              # redis | memcached | in-memory
  # in-memory is incompatible with infrastructure.scalability.horizontal: true

  default-ttl-seconds: 300     # used when caller does not specify TTL

  max-connections: 10          # connection pool size (redis only)
```
