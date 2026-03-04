// will write teh cotentn near future (take refferene from the system design school)

# Token Bucket Rate Limiting with Redis (Summary)

## Core Idea
- We usually do **lazy refill**: tokens are recalculated when a request arrives.
- We do **not** need to add tokens at fixed intervals in a background scheduler.

## Per-User Bucket State in Redis
- `tokens`: current available tokens.
- `last_refill_ts`: last time refill was calculated.

## Request-Time Flow
1. Read current time (`now`).
2. Compute elapsed time: `elapsed = now - last_refill_ts`.
3. Compute new tokens: `new_tokens = elapsed * refill_rate`.
4. Refill with cap: `tokens = min(capacity, tokens + new_tokens)`.
5. If `tokens >= 1`:
- allow request
- decrement `tokens = tokens - 1`
6. Else:
- reject with rate-limit response (for example `429`).
7. Save updated `tokens` and `last_refill_ts` back to Redis.

## Race Condition Handling
- Use a **single Redis Lua script** for refill + check + decrement + save.
- Lua execution in Redis is atomic, so concurrent requests cannot interleave this logic.
- This prevents double-consume and inconsistent token state.

## Alternative (Less Preferred)
- `WATCH` + `MULTI/EXEC` with retries can work, but is more complex and can conflict under high contention.

## Operational Note
- Set key TTL (`EXPIRE`) so inactive user buckets are automatically cleaned up.

