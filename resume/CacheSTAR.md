# Redis Token Caching — DB Contention Deep Dive

Reference notes on how database contention occurs in a stateful (opaque token) auth flow, and how introducing a Redis cache layer resolves it. Useful for interview prep (STAR method) and as architecture documentation.

---

## Context

In a stateful auth flow, every authenticated request needs to resolve an opaque access token to a user ID. Without caching, this means a database query (`SELECT ... FROM oauth_accs_tkn WHERE access_token = ?`) on **every single request** — not just at login. Since a user typically makes hundreds of authenticated calls per session, this hammers the database continuously.

**Contention** is competition for a finite shared resource. In this system, several resources get contended for simultaneously.

---

## 1. Connection Pool Exhaustion

- The app has a finite connection pool (e.g. 20–50 connections via HikariCP).
- Every operation — including a simple token lookup — must borrow a connection, use it, and return it.
- Without Redis: every authenticated request borrows a connection just to validate a token.
- Meanwhile, other business logic (`orders` inserts, `inventory` updates, etc.) competes for the *same* pool.
- If all connections are busy, new requests — including important writes — **queue and wait** for a free connection. That queuing delay is contention.

## 2. Row / Index Lock Contention

- Even read queries interact with locking depending on isolation level.
- `oauth_accs_tkn` is read constantly while also being written to (new logins, refreshes, deletes on logout) → **read-write contention** on the same table/index.
- Under higher isolation levels, a read can briefly block on a row that's mid-write (e.g. a logout deleting that exact token row at the same moment).
- The index on `access_token` is a **hot index** — high-frequency access means more internal latching/locking at the storage engine level, even for pure reads.

## 3. I/O and CPU Contention

- The DB engine has finite CPU cores and disk I/O throughput.
- Every query — even a fast indexed lookup — consumes CPU cycles to parse, plan (or reuse a cached plan), execute, and serialize a result.
- At high frequency (thousands of token-validation queries per minute), that CPU/I/O time is unavailable for other queries — analytics, reporting, order processing, etc.
- Net effect: it's not just "the token query is slow" — it's "the token query, multiplied by volume, is making *every other query* on the DB slower too," since they're all fighting for the same execution resources.

## 4. Cache / Buffer Pool Pollution

- Databases keep frequently-accessed data in memory (buffer pool / page cache) to avoid disk reads.
- Constant hammering of `oauth_accs_tkn` keeps its pages "hot" in the buffer pool — which means **less memory available** for caching pages from other tables.
- Other queries that would've been served from memory now hit disk more often — a subtler, second-order form of contention.

---

## Why Redis Fixes This

Redis doesn't just make the token query itself faster — it **removes that query from the DB's workload almost entirely**.

| Aspect | Without Redis | With Redis |
|---|---|---|
| Latency | 10–50ms (DB query) | ~1ms (Redis GET) |
| DB Load | Every request | Only on cache miss |
| Scalability | DB becomes bottleneck | Horizontal scaling via Redis Cluster |

- Token metadata is cached on login: `SET <access_token> <user_id> EX <ttl>` — TTL matches token lifetime, so Redis handles expiry automatically.
- On each request: Redis `GET` first (fast path, ~1ms). On a miss, fall back to DB, then re-populate Redis so the cache stays warm without a separate warming job.
- On logout/refresh: `DEL` the key explicitly, so revoked tokens don't stay valid in cache (TTL alone isn't enough — the token may still have time left on its natural expiry).
- DB is instead only touched on a cache miss — first request after login, or after a Redis eviction/restart — which is why overall DB load drops dramatically (~90% in this case), not just the auth path's own latency.

**Design principle:** DB is the source of truth; Redis is a disposable accelerator. Every write path (login, logout, refresh) updates the DB first. Redis failures are caught and swallowed rather than failing the request — losing the cache degrades performance, not correctness.

---

## Interview-Ready Summary

> "Contention wasn't just the token query being slow on its own — it was that every token check consumed a DB connection, some CPU/IO time, and touched a hot index, all of which competed with the rest of the app's queries for the same limited resources. At high request volume, that queuing and competition for connections/locks/CPU was dragging down response times system-wide, not just for auth. Moving token lookups to Redis removed that traffic from the DB almost entirely, freeing up connections and DB resources for everything else."

---

## Related Follow-Up Topics to Prep

- Connection pool sizing (e.g. HikariCP) and how pool size relates to contention
- Cache stampede / thundering herd mitigation (TTL jitter, single-flight locking)
- Redis Cluster vs single-node trade-offs
- Consistency model: why DB-first writes + disposable cache avoids split-brain between DB and cache
