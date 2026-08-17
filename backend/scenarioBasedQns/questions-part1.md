# Backend Development — Scenario-Based Interview Notes (Standard Approaches)

## Q1: p99 latency spike (200ms → 3s), CPU/memory normal
1. Check what changed at that moment: deploy, traffic spike, cron job, downstream dependency degraded.
2. Use distributed tracing (trace ID) to find which service/hop is slow.
3. Check DB query time/execution plan, missing indexes.
4. Check connection pool exhaustion, GC pauses (common cause of high latency with normal CPU).
5. Check synchronous I/O calls that could be made async/event-driven.
6. Fix root cause → deploy → monitor.

---

## Q2: Order placement calls 3 external APIs (payment, inventory, notification); notification is slow/unreliable
- **Payment & inventory**: synchronous, critical path (must succeed before confirming order) — protect with timeout + circuit breaker + retry with exponential backoff.
- **Notification**: fully asynchronous via message queue/event — fire-and-forget, never blocks order flow.
- General rule: sync for what must succeed before responding to the user, async/queue for what can happen after.

---

## Q3: Order Service + Inventory Service, separate DBs, consistency without 2PC
- Use **Saga pattern**:
  - **Choreography saga**: services react to each other's events, no central controller.
  - **Orchestration saga**: central coordinator tells each service what to do — better for complex multi-step flows.
- Each service does a local transaction + publishes an event.
- On failure, run a **compensating transaction** (e.g., inventory fails → publish `OrderCancelled` → order service rolls back/cancels the order).

---

## Q4: Large file uploads (500MB+) causing timeouts/OOM
- Never route file bytes through the app server.
- Use **S3 pre-signed URLs / multipart upload** — client uploads chunks directly to S3.
- App server only handles metadata: chunk ID, sequence, hash, status — stored in DB.
- Support **resumable uploads** and a completion webhook to trigger post-processing.

### Presigned URL flow
- **Presigned URL** = real HTTPS URL pointing at S3 with a temporary signed authorization (SigV4: `X-Amz-Signature`, `X-Amz-Expires`, etc.) in the query params. Holder can PUT to that exact key until expiry — no AWS credentials needed client-side.
- `getSignedUrl(s3, command, { expiresIn: 600 })` — pure local HMAC-SHA256 signing using the server's AWS secret key. No network call to AWS. Secret key never leaves the server.
- Naming: `uploadId` (UUID, server-generated, groups all chunks) + `chunkNumber` (sequential) → S3 key `uploads/{uploadId}/chunk-{n}`.
- Optimize multiple chunk uploads via:
  1. One batch API call returning all N presigned URLs at once.
  2. **S3 Multipart Upload API** — `CreateMultipartUpload` once, presigned URL per part under that upload ID, `CompleteMultipartUpload` at the end. Native parallel upload + resume support.

**End-to-end flow:**
1. Client → `POST /upload/init` → server generates `uploadId`, N presigned URLs, inserts DB rows (`uploads`, `upload_chunks`, status=pending).
2. Client PUTs each chunk directly to S3 using its presigned URL.
3. Client → `POST /upload/chunk-complete` per chunk → server updates chunk status.
4. Client → `POST /upload/complete` → server verifies all chunks uploaded → finalizes.

---

## Q5: Single shared MySQL DB, bottleneck under read + write heavy traffic
Order of scaling options:
1. **Vertical scaling** (bigger instance) — cheapest, no code change.
2. **Query/index optimization + connection pooling** — fix inefficiency before adding infra.
3. **Caching layer** (Redis) for hot reads.
4. **Read replicas** — scale reads, route SELECTs to replicas.
5. **Sharding/partitioning** — scale writes; split data horizontally across multiple primaries by shard key (e.g., `user_id % N`).
6. Consider a different DB engine/architecture only if MySQL genuinely doesn't fit the access pattern.

(Primary-standby replication solves **availability/failover**, not write scaling — don't conflate the two.)

---

## Q6: "Get Order Details" API, called millions of times/day, data rarely changes
- **Cache-aside pattern**: on request, check cache first; on miss, fetch from DB, populate cache, return.
- Use **TTL** (e.g., 1 hour) and/or **event-driven invalidation** (on order update, invalidate/update the cache entry).
- Use a **distributed cache** (Redis/Memcached) shared across all app instances — not per-instance in-memory.
- Cache the **serialized JSON response** directly under key `order:{orderId}` to skip re-serialization.
- Route cache misses to a **read replica**, not the primary write DB.
- Optionally add **CDN/edge caching** if the response is public/static-ish per order ID.

---

## Q7: Rate limiter — 100 requests/user/minute
- **Fixed window**: simple, but allows burst at window boundaries.
- **Sliding window log**: accurate, stores every request timestamp — accurate but storage-expensive.
- **Sliding window counter**: hybrid — weights previous window's count proportionally; cheaper than full log.
- **Token bucket**: tracks remaining tokens + last refill timestamp; allows controlled bursts; industry-standard balance of accuracy and cost.
- **Leaky bucket**: smooths bursts into a constant output rate; used often at gateway level.
- **Storage**: use Redis (atomic `INCR` + `EXPIRE`) for fast, shared, atomic counters across distributed app servers.
- **Enforcement layer**: API Gateway/Nginx/Kong (centralized) vs application level (more flexible, per-endpoint rules).

---

## Q8: REST API + real-time updates — WebSocket vs SSE vs polling
- **SSE (Server-Sent Events)**: correct choice for one-directional (server → client) push. Native browser auto-reconnect (`EventSource`), works over plain HTTP/1.1, simpler infra. Limitations: ~6 concurrent connections per domain (HTTP/1.1), text-only, one-way only.
- **WebSocket**: use only when bidirectional communication is needed (chat, collaborative editing) — otherwise overkill; needs stateful connection and extra infra handling (LB/proxy/CDN config).
- **Polling**: short-polling — simplest, but adds delay/staleness and unnecessary load. Long-polling — middle ground, connection held open until new data or timeout.

---

## Q9: Idempotent payment API — prevent double charge on retry
- Client generates an **idempotency key** (UUID) once, before the request, sent as a header (`Idempotency-Key`). Must be client-generated so a retry after timeout reuses the same key.
- Server flow:
```javascript
app.post("/charge", async (req, res) => {
  const idempotencyKey = req.headers["idempotency-key"];
  const existing = await db.query(`SELECT * FROM idempotency_keys WHERE key = ?`, [idempotencyKey]);
  if (existing) return res.status(existing.status_code).json(existing.response_body);

  const result = await chargeCard(req.body);
  await db.query(
    `INSERT INTO idempotency_keys (key, response_body, status_code) VALUES (?, ?, ?)`,
    [idempotencyKey, JSON.stringify(result), 200]
  );
  res.json(result);
});
```
- Check-and-insert must be **atomic** (unique constraint on key column, or distributed lock) to prevent race conditions on simultaneous retries.
- Idempotency keys should have a **TTL** (e.g., 24h) to avoid unbounded table growth.

---

## Q10: Kafka consumer updates DB, crashes before committing offset, reprocesses event
- Use the **idempotent-consumer pattern**: track processed event IDs in the DB.
- Do the business update **and** the "mark event processed" insert in the **same DB transaction**:
```sql
BEGIN;
UPDATE orders SET status='charged' WHERE id=?;
INSERT INTO processed_events (event_id) VALUES (?);
COMMIT;
```
- Commit the Kafka offset only **after** this transaction succeeds.
- On reprocessing, check `processed_events` for the event ID; if present, skip the side effect.
- This combination (Kafka's at-least-once delivery + idempotent processing) yields **effectively-once** semantics.
- Avoid trying to wrap DB + Kafka offset commit in a single distributed transaction — inefficient and unnecessary.

---

## Q11: `/search` API using `LIKE '%keyword%'` on millions of rows — slow
- Leading wildcard (`%keyword%`) defeats B-tree indexes entirely → full table scan. (Trailing wildcard `keyword%` can still use an index.)
- **Lightweight fix**: MySQL `FULLTEXT` index + `MATCH() AGAINST()` — no new infra, good for whole-word search.
- **Standard fix for scale/partial-match search**: move search to **Elasticsearch**.
  - Uses an **inverted index** — maps tokens to document IDs instead of scanning documents.
  - Use **n-gram tokenization** for partial/substring/mid-word matching.
- **Sync concern**: MySQL remains source of truth, ES is a derived index — keep them in sync via **CDC** (e.g., Debezium) or an outbox pattern.

---

## Q12: Zero-downtime deployment with instant rollback
- **Canary deployment**: deploy V2 alongside V1; route a small % of traffic (e.g., 5%) to V2 via load balancer/service mesh (Istio) weighted routing or API Gateway rules.
- Monitor error rates, p99 latency, and business metrics before increasing V2's traffic share.
- Gradually increase V2 traffic to 100% if healthy; keep V1 running throughout.
- **Instant rollback** = shift traffic back to 100% V1 via routing config change — no redeploy needed.
- Decommission V1 only after V2 is fully validated at 100% traffic.
- **Alternative**: Blue-Green deployment — full V2 environment provisioned in parallel, traffic switched all-at-once via router/DNS; instant rollback by switching back. Faster cutover, but risk only surfaces at full scale (no gradual exposure).

---

## Q13: Secure password storage — signup to login
- **Never store plaintext or use reversible encryption** for passwords.
- Use a purpose-built password hashing algorithm: **bcrypt, scrypt, or Argon2 (Argon2id preferred)** — these handle salting internally and automatically.
- Passwords are **hashed (one-way)**, never encrypted/decrypted.
- Transport security is handled by **HTTPS/TLS** — no client-side "encryption" (e.g., base64) is needed or effective; base64 is encoding, not security.

```javascript
// Signup
const hash = await bcrypt.hash(plainPassword, 12);
await db.query(`INSERT INTO users (email, password_hash) VALUES (?, ?)`, [email, hash]);

// Login
const user = await db.query(`SELECT password_hash FROM users WHERE email=?`, [email]);
const isValid = await bcrypt.compare(plainPasswordFromLogin, user.password_hash);
if (isValid) { /* issue session/JWT */ }
```

---

## Q14: JWT logout, expiry, and refresh (JWT is stateless)
- **Access token**: short-lived (5–15 min), stateless JWT, signature-verified on each request. Cannot be revoked directly since it's stateless — short expiry limits the exposure window.
- **Refresh token**: long-lived, stored server-side (DB row: `user_id`, `device_id`, `expires_at`) — this is the part that IS revocable.
- **Login**: issue both an access token and a refresh token.
- **Logout**: delete the refresh token's DB row → client can no longer obtain new access tokens. The current (already-issued) access token remains valid until it naturally expires — an accepted trade-off given the short expiry.
- **Immediate revocation (optional, if required)**: maintain a **token blocklist/denylist in Redis**, storing revoked access token IDs (`jti` claim) with TTL = remaining token lifetime; check against this list on each request.
- **Refresh flow**: client sends refresh token when access token expires → server checks the DB row exists and is unexpired → issues new access token; otherwise return **401 Unauthorized**.

---

# Reference: Zero-Downtime DB Schema Migration Strategy (Expand–Migrate–Contract)

## The Core Problem
During rolling deployment, two app versions run simultaneously against the same DB. Schema must work for both. Never combine schema change + code change in one deployment.

## Phase 1: Expand (additive, backward-compatible)
- Add new column as **nullable** or with **DEFAULT** — never `NOT NULL` without default.
- Old code ignores new column entirely.
```sql
ALTER TABLE transactions ADD COLUMN payment_channel VARCHAR(50) NULL DEFAULT 'UNKNOWN';
```

## Phase 2: Dual-Write / Backfill
- New code writes to **BOTH** old and new column/structure (not just the new one) — keeps old-code instances (still running) consistent.
- Batched backfill for historical rows:
```sql
UPDATE transactions SET payment_channel = 'LEGACY' WHERE payment_channel IS NULL LIMIT 1000;
```
- Reads: new code reads new column, falls back to old column if null.

## Phase 3: Contract (cleanup)
- Once all instances on new code + backfill verified: deploy code that only reads/writes new column.
- Only then add `NOT NULL` constraint or drop old column — in a separate later migration.

## Key Talking Points
| Practice | Why |
|---|---|
| Never combine schema + code deploy | Both versions hit DB simultaneously during rollout |
| Additive-only in expand phase | No renames/drops — add new, migrate, then remove old |
| Feature flags | Toggle behavior without redeploy; safe rollback |
| Forward compatibility | Rolled-back old code must not break on new schema |
| Batched backfills | Avoid table locks & replication lag |
| Idempotent migration scripts | Safe to re-run (Flyway/Liquibase/Alembic) |

## New Transactions During Migration Window
- New rows inserted after column added but before all nodes updated → get default value, reconciled later via backfill/dual-write.
- Danger: `NOT NULL` column **without** default while old code still inserts → insert failures.
- Rule: nullable/default first, tighten constraints only in contract phase.
