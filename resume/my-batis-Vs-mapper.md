# MyBatis vs JPA/Hibernate — Tradeoff Reference

A quick-reference comparison for explaining ORM migration decisions in interviews or design discussions.

---

## Core Philosophy

| | MyBatis | JPA/Hibernate |
|---|---|---|
| **Approach** | SQL Mapper — you write SQL, it maps results to Java objects | Full ORM — you define entities, it generates SQL |
| **Control** | Full control over exact SQL executed | Declarative; SQL is generated (can override with JPQL/native queries) |
| **Boilerplate** | High — mapper XML/annotations + SQL per query | Low — `JpaRepository` gives CRUD for free |
| **Learning curve** | Lower if you already know SQL | Higher — need to understand entity lifecycle, sessions, caching |
| **Best fit** | Complex reporting queries, fine-tuned performance-critical SQL, teams that want full SQL visibility | Standard CRUD-heavy apps, rapid feature development, relationship-heavy domain models |

---

## Why teams migrate MyBatis → JPA/Hibernate

- Repetitive CRUD boilerplate — every entity needs its own mapper XML + hand-written `INSERT`/`UPDATE`/`SELECT`
- No automatic relationship management (joins, cascades) — manually written each time
- No dirty checking — must manually track and write update statements
- Slower feature velocity as the number of entities grows
- Harder to keep query logic DRY across similar entities

**Typical outcome quoted:** ~30% less boilerplate code, ~40% faster feature delivery (new entity = repository interface + annotations, not hand-written SQL + mapper + DAO).

---

## What you gain with JPA/Hibernate

- `JpaRepository` — CRUD methods for free (`save`, `findById`, `deleteById`, derived query methods like `findByEmail`)
- Automatic dirty checking — no manual `UPDATE` statements for tracked entity changes
- Relationship mapping (`@OneToMany`, `@ManyToOne`, etc.) with cascade/fetch control
- First & second-level caching built in
- Less code to write and maintain for standard CRUD operations

## What you give up / must watch for

| Risk | Description | Mitigation |
|---|---|---|
| **N+1 query problem** | Lazy-loaded associations can silently fire one query per row instead of a single join | `JOIN FETCH`, `@EntityGraph`, batch fetching |
| **Less control over generated SQL** | Auto-generated SQL can be inefficient for complex reporting/analytics queries | Use native queries or JPQL `@Query` for performance-critical paths |
| **`LazyInitializationException`** | Accessing a lazy association outside an active transaction/session fails | Use `@Transactional` boundaries carefully, or fetch eagerly/via `JOIN FETCH` where needed |
| **Dirty-checking overhead** | Entity state tracking has memory/CPU cost, noticeable in high-throughput batch operations | Use stateless sessions or batch/bulk operations for large writes |
| **Hidden transaction/flush behavior** | Hibernate's session flush timing differs from MyBatis's explicit statement execution | Understand flush modes; be explicit about transaction boundaries |

---

## Migration approach (talking points)

- Prefer **incremental migration** (module by module or entity by entity) over a big-bang rewrite — lower risk, easier rollback
- Keep native/JPQL queries for reporting or performance-sensitive paths instead of forcing everything through auto-generated JPQL
- Watch for behavioral differences during migration:
  - Null vs missing row handling
  - Transaction boundary differences (Hibernate session/flush vs MyBatis's explicit control)
  - Lazy loading exceptions when entities are touched outside a transaction

---

## Interview-ready summary (30-45 sec version)

> "Our MyBatis layer required hand-written SQL and mapper XML for every query, which meant a lot of repetitive boilerplate as the number of entities grew. I led the migration to JPA/Hibernate, using Spring Data repositories for standard CRUD and JPQL/native queries where we needed more control — particularly for reporting queries where auto-generated SQL wasn't efficient enough. This cut boilerplate by about 30%, since most simple queries became a repository interface method instead of hand-written mapper code, and sped up feature delivery by roughly 40% since adding a new entity or endpoint no longer required writing SQL and mapping code from scratch. The main thing I had to watch for was the N+1 query problem with lazy loading, which we addressed with `@EntityGraph` and fetch joins on hot paths."

**One-liner version:**
> "MyBatis gives full SQL control at the cost of boilerplate; JPA/Hibernate trades some of that control for a much faster development loop via automatic CRUD, dirty checking, and relationship mapping — the main risk to manage is the N+1 query problem and understanding Hibernate's session/flush lifecycle."

---

## Likely interviewer follow-ups (be ready)

1. "Doesn't Hibernate have performance risks MyBatis doesn't?" → N+1, generated SQL inefficiency, dirty-checking overhead
2. "How did you handle the migration itself?" → incremental vs big-bang, coexistence strategy
3. "Any data integrity or behavioral bugs during migration?" → null handling, transaction boundaries, lazy-loading exceptions
4. "When would you still use MyBatis/native SQL over JPA?" → complex reporting, bulk operations, performance-critical read paths
