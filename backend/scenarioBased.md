# Zero-Downtime DB Schema Migration Strategy
## (Expand–Migrate–Contract Pattern)

## The Core Problem
During a rolling deployment, **two versions of app code run simultaneously** against the **same database** (old pods shutting down + new pods starting up). The schema must work for **both versions at once**. Never combine a schema change and a code change into a single deployment.

---

## Phase 1: Expand (Additive, Backward-Compatible)
- Add the new column as **nullable** or with a **DEFAULT value** — never `NOT NULL` without a default.
- Old app code ignores the new column entirely → nothing breaks.
- Deploy this schema change **alone**, with no app code changes yet.

```sql
ALTER TABLE transactions 
ADD COLUMN payment_channel VARCHAR(50) NULL DEFAULT 'UNKNOWN';
```

---

## Phase 2: Dual-Write / Backfill
- Deploy new app code that:
  - Writes to **both** old and new column/logic (dual write)
  - Reads with **fallback logic** if new column is null (for old rows)
- Run a **batched backfill job** (throttled, off-peak) to populate historical rows:

```sql
UPDATE transactions 
SET payment_channel = 'LEGACY' 
WHERE payment_channel IS NULL 
LIMIT 1000; -- loop in batches, not one giant update
```

- App must tolerate 3 states during this window:
  1. Old rows (null/default)
  2. Backfilled rows
  3. New rows written by new code

---

## Phase 3: Contract (Cleanup)
- Once **all instances** run new code AND backfill is verified complete:
  - Deploy code that only reads/writes the new column
  - Optionally add `NOT NULL` constraint or drop old column — **in a separate, later migration**

---

## Key Interview Talking Points

| Practice | Why it matters |
|---|---|
| Never combine schema + code deploy | Both old & new code hit DB simultaneously during rollout |
| Additive-only changes in expand phase | No renames/drops — always add new, migrate, then remove old |
| Feature flags | Toggle new column read/write without redeploying; safe rollback |
| Forward compatibility | Old code (during rollback) must not break on new schema |
| Batched backfills | Avoids table locks & replication lag on large tables |
| Idempotent migration scripts | Safe to re-run (Flyway/Liquibase/Alembic) |
| Verification between phases | Check null %, row counts, error rates before contract phase |

---

## Answering the "New Transactions" Concern
- New transactions inserted **after** column is added but **before** all nodes are updated → fine, since they get the **default value**.
- These get reconciled later via backfill job or dual-write logic.
- **Danger case:** Adding `NOT NULL` column **without** a default while old code is still inserting → causes insert failures.
- **Rule:** Always nullable/default first → tighten constraints only in the final contract phase.

---

## Summary Flow
```
Expand (add column, nullable/default)
   ↓
Migrate (dual-write + backfill old rows)
   ↓
Contract (cleanup, drop old logic, tighten constraints)
```
