# DB Partitioning vs DB Sharding

## 1. DB Partitioning

**Partitioning = splitting a large table into smaller partitions, usually within the same database system.**

### Example

Suppose we have a huge `orders` table:

```text
                    PostgreSQL
                        |
                   orders table
                        |
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
      orders_2024   orders_2025   orders_2026
```

We can partition the table based on `created_at`.

```sql
CREATE TABLE orders (
    id BIGINT,
    created_at DATE,
    amount DECIMAL
) PARTITION BY RANGE (created_at);
```

### When to use partitioning?

Use partitioning when:

* A single table becomes very large.
* One database server can still handle the workload.
* You want better query performance through partition pruning.
* You want easier management/deletion of historical data.
* Data naturally belongs to ranges/categories.

### Common partitioning strategies

#### Range Partitioning

```text
2024 → Partition 1
2025 → Partition 2
2026 → Partition 3
```

Good for:

* Dates
* Time-series data
* Sequential IDs

#### List Partitioning

```text
India  → Partition 1
USA    → Partition 2
Germany → Partition 3
```

Good when data belongs to known categories.

#### Hash Partitioning

```text
hash(user_id) % 4

0 → Partition 1
1 → Partition 2
2 → Partition 3
3 → Partition 4
```

Good for distributing data relatively evenly.

---

# 2. DB Sharding

**Sharding = distributing data across multiple independent database servers/instances.**

### Example

Suppose we have 500 million users and one database cannot handle the workload.

We can shard using `user_id`:

```text
                    Application
                         |
             ┌───────────┼───────────┐
             ↓           ↓           ↓
          DB Shard 1  DB Shard 2  DB Shard 3
          Users       Users       Users
          1 - 166M    166-333M    333-500M
```

Or using hash-based sharding:

```text
user_id % 4

0 → Shard 1
1 → Shard 2
2 → Shard 3
3 → Shard 4
```

### When to use sharding?

Use sharding when:

* One database server cannot handle the workload.
* Database storage is too large.
* CPU/IOPS/write throughput is saturated.
* You need horizontal database scaling.
* You need to distribute traffic across multiple database machines.

---

# 3. Partitioning vs Sharding

| Feature               | Partitioning                | Sharding                        |
| --------------------- | --------------------------- | ------------------------------- |
| What is split?        | Large table/data            | Database data                   |
| DB servers            | Usually one                 | Multiple                        |
| Main goal             | Performance & manageability | Horizontal scalability          |
| Complexity            | Lower                       | Higher                          |
| Application awareness | Usually low                 | Often needs shard routing       |
| Example               | Orders by year              | Users by `user_id`              |
| Scaling               | Mostly vertical             | Horizontal                      |
| Cross-data queries    | Relatively easy             | Can require distributed queries |

### Easy way to remember

```text
Partitioning
    ↓
"One DB, huge table"
    ↓
Split the table


Sharding
    ↓
"One DB server is not enough"
    ↓
Split data across DB servers
```

> **Partitioning = divide the table.**
> **Sharding = divide the database across machines.**

---

# 4. Partitioning Example

Suppose:

```text
orders = 1 billion rows
```

But our database server has enough:

* CPU
* RAM
* Storage
* IOPS
* Network capacity

Then we can partition:

```text
                    DB Server
                       |
                     orders
                       |
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      2024            2025            2026
```

Query:

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-01-01';
```

The database can potentially scan only the relevant partition instead of the entire table.

This is called **partition pruning**.

---

# 5. Sharding Example

Suppose:

```text
500 million users
100K writes/sec
One DB server is overloaded
```

We can distribute users:

```text
                    Application
                         |
             ┌───────────┼───────────┐
             ↓           ↓           ↓
          Shard 1      Shard 2      Shard 3
          Users        Users        Users
          1-166M       166-333M     333-500M
```

Now the workload is distributed across multiple database servers.

### Shard Key

A **shard key** determines which shard contains the data.

Example:

```text
user_id
```

```text
hash(user_id) → shard
```

Choosing a good shard key is very important.

A bad shard key can create a **hot shard**.

Example:

```text
India → 80% of users ❌
USA   → 10%
Other → 10%
```

If we shard only by country, the India shard could become overloaded.

---

# 6. Can We Use Both?

Yes.

A production system can use:

**Sharding + Partitioning + Replication**

Example:

```text
                       Application
                            |
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
           Shard 1       Shard 2       Shard 3
              |             |             |
              ↓             ↓             ↓
          Partition     Partition     Partition
          by month      by month      by month
```

Each shard can independently partition its tables.

This is useful when:

* The total dataset is huge → **Sharding**
* Each shard's tables are also huge → **Partitioning**

---

# 7. Partitioning vs Replication vs Sharding

These solve different problems.

### Partitioning

```text
One DB
   ↓
Split large table
```

Main goal:

**Query performance + data management**

---

### Replication

```text
              Primary
             /       \
            ↓         ↓
       Replica 1   Replica 2
```

Copies the **same data** to multiple servers.

Main goals:

* High availability
* Read scaling
* Disaster recovery

---

### Sharding

```text
DB-1 → Users 1-1M
DB-2 → Users 1M-2M
DB-3 → Users 2M-3M
```

Each database contains **different data**.

Main goal:

**Horizontal scalability**

---

# 8. System Design Decision

When you have a large database, don't immediately choose sharding.

Ask:

```text
                 Is the table very large?
                          |
                         YES
                          |
                          ↓
              Can one DB server handle it?
                    /             \
                  YES              NO
                   |                |
                   ↓                ↓
             PARTITIONING       SHARDING
```

### Scenario 1: Large historical data

```text
500 million orders
One DB can handle workload
```

Use:

**Partitioning**

```text
orders
├── 2024
├── 2025
└── 2026
```

---

### Scenario 2: Massive user base + high traffic

```text
500 million users
Very high read/write traffic
One DB is overloaded
```

Use:

**Sharding**

```text
user_id
   ↓
Shard 1
Shard 2
Shard 3
Shard 4
```

---

### Scenario 3: Extremely large dataset + high traffic

```text
5 billion orders
100K+ writes/sec
```

Potential architecture:

```text
                Application
                     |
           ┌─────────┼─────────┐
           ↓         ↓         ↓
        Shard 1   Shard 2   Shard 3
           |         |         |
           ↓         ↓         ↓
       Partition  Partition  Partition
       by month   by month   by month
```

Use:

**Sharding + Partitioning**

---

# 9. Interview Answer

### What is DB partitioning?

> **DB partitioning divides a large table into smaller partitions within a database system. It is mainly used to improve query performance, manageability, and maintenance of large tables.**

### What is DB sharding?

> **DB sharding distributes data across multiple independent database instances or servers. It is primarily used for horizontal scalability when a single database cannot handle the required storage or workload.**

### What is the key difference?

> **Partitioning solves the problem of a large table, while sharding solves the problem of a database server that cannot handle the required scale.**

### Quick memory trick

```text
PARTITIONING
    ↓
Large TABLE
    ↓
Split table


SHARDING
    ↓
Large DATABASE / high workload
    ↓
Split across MACHINES


REPLICATION
    ↓
Need HA / read scaling
    ↓
Copy data across machines
```

## Final takeaway

```text
Large table
    ↓
Can one DB handle it?
    ├── YES → Partitioning
    └── NO  → Sharding

Need high availability/read scaling?
    ↓
Add Replication

Extremely large + high traffic?
    ↓
Sharding + Partitioning + Replication
```
