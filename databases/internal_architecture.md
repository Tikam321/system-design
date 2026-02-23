# How SQL and NoSQL (Cassandra & DynamoDB) Work Internally

---

# 1. Relational Databases (SQL) – Internal Working

Examples:
- MySQL
- PostgreSQL
- Oracle

Relational databases are typically optimized for:
- Strong consistency
- ACID transactions
- Balanced read/write workloads

---

## 1.1 Core Storage Structure

Most SQL databases use:

- **B-Tree / B+ Tree** indexes

### Why B-Tree?

B-Trees:
- Keep data sorted
- Allow O(log N) search
- Minimize disk reads
- Efficient for range queries

---

## 1.2 Data Organization

```text
Table
  ↓
Primary Key Index (B+ Tree)
  ↓
Disk Pages (8KB/16KB)
```

Each table is stored as:
- A collection of pages
- Organized using a B+ Tree

---

## 1.3 Insert Operation (SQL)

When inserting a row:

1. Traverse B+ Tree → O(log N)
2. Locate correct leaf page
3. Insert record
4. If page is full → split page
5. Update parent nodes

This causes:
- Random disk writes
- Page splits
- Tree rebalancing

---

## 1.4 Read Operation (SQL)

For primary key lookup:

1. Traverse B+ Tree from root
2. Reach leaf node
3. Fetch row

Time complexity:

O(log N)

For range queries:
- Very efficient
- Because data is sorted

---

## 1.5 Write-Ahead Logging (WAL)

Before modifying disk:

- SQL databases write to WAL (Write Ahead Log)
- Ensures durability
- If crash occurs → replay WAL

---

## 1.6 Characteristics of SQL Storage

- Random disk writes
- In-place updates
- Strong consistency
- Optimized for structured queries
- Good for joins and transactions

---

# 2. Cassandra – Internal Working

Cassandra is based on:

- Google Bigtable
- Amazon Dynamo

It uses:

**LSM Tree (Log Structured Merge Tree)**

---

## 2.1 Why LSM Instead of B-Tree?

B-Tree:
- Random writes
- Expensive disk IO

LSM:
- Sequential writes
- High write throughput
- Better for distributed systems

---

## 2.2 Cassandra Write Path

When writing data:

```text
Client
  ↓
Coordinator Node
  ↓
Commit Log (disk)
  +
MemTable (RAM)
```

### Step 1 – Commit Log
- Append-only file
- Sequential write
- Ensures durability

### Step 2 – MemTable
- In-memory structure
- Sorted by partition key
- Fast writes

---

## 2.3 MemTable Flush

When MemTable is full:

```text
MemTable → Flushed → SSTable
```

SSTable = Sorted String Table

- Immutable
- Sorted on disk
- Written sequentially

After flush:
- Commit log segment deleted
- MemTable cleared

---

## 2.4 Cassandra Read Path

When reading:

1. Check MemTable
2. Check Bloom Filter (per SSTable)
3. Check Partition Index
4. Read from SSTable

---

## 2.5 SSTable Structure

Each SSTable contains:

- Data file
- Partition index
- Bloom filter
- Summary index

Lookup inside SSTable:

Binary search → O(log N)

---

## 2.6 Updates in Cassandra

Cassandra does NOT update in place.

Instead:

- Writes a new version with higher timestamp
- Old version remains
- Compaction removes old version later

---

## 2.7 Deletes in Cassandra

Deletes create:

- Tombstones

Removed during compaction.

---

## 2.8 Compaction

Background process that:

- Merges SSTables
- Removes old versions
- Reduces read amplification

---

## 2.9 Data Distribution

Cassandra uses:

**Consistent Hashing Ring**

Partition key → Murmur3 hash → Token range → Node

Each node owns a token range.

Replication:
- Data replicated to multiple nodes
- Configurable replication factor

---

## 2.10 Cassandra Characteristics

- High write throughput
- Eventually consistent (configurable)
- Hash-based partitioning
- No efficient global range scan
- LSM-based storage

---

# 3. DynamoDB – Internal Working

DynamoDB is based on:

Amazon Dynamo Paper.

Core concepts:

- Consistent hashing
- Partition-based storage
- Log-structured storage
- Eventually consistent replication

---

## 3.1 Partitioning

Partition Key → Hash → Physical Partition

Each partition:
- Stored across multiple machines
- Replicated across Availability Zones

---

## 3.2 Write Path (Dynamo-style)

1. Request routed to correct partition
2. Write appended to log
3. Stored in in-memory structure
4. Flushed to storage

Similar to LSM-based systems.

---

## 3.3 Replication

Uses:

- Multi-AZ replication
- Leader-based or quorum-based writes

Supports:
- Eventually consistent reads
- Strongly consistent reads (optional)

---

## 3.4 Scaling

DynamoDB:
- Automatically splits partitions
- Based on throughput or size
- No manual sharding required

---

## 3.5 Indexing

Supports:
- Primary index (partition key + sort key)
- Global Secondary Index (GSI)
- Local Secondary Index (LSI)

Internally uses:
- Log-structured storage
- Hash-based routing

---

# 4. Key Differences – SQL vs Cassandra vs DynamoDB

| Feature | SQL (B-Tree) | Cassandra (LSM) | DynamoDB |
|----------|--------------|----------------|-----------|
| Storage Structure | B+ Tree | LSM Tree | LSM-like |
| Write Type | Random | Sequential | Sequential |
| Update | In-place | Append-only | Append-like |
| Read Complexity | O(log N) | O(K × log N) | O(1) partition + local lookup |
| Partitioning | Vertical scaling | Consistent hashing | Hash partition |
| Transactions | Strong ACID | Limited | Limited |
| Best For | Complex queries | Write-heavy workloads | Massive scale |

---

# 5. Conceptual Summary

## SQL

- Single machine optimized
- Balanced read/write
- Strong consistency
- Random IO

---

## Cassandra

- Distributed ring
- Write optimized
- LSM tree
- Append-only
- Compaction-based cleanup

---

## DynamoDB

- Managed distributed system
- Hash-based routing
- Auto partition scaling
- High availability

---

# 6. Final Mental Model

SQL:
- Sorted tree
- O(log N)
- Random writes

Cassandra:
- Append log
- In-memory buffer
- Immutable SSTables
- Compaction later

DynamoDB:
- Partition by hash
- Log structured storage
- Replication across regions

---

# 7. Interview-Level Explanation

If asked:

"How does Cassandra differ from MySQL internally?"

You can answer:

"MySQL uses B+ Trees with in-place updates leading to O(log N) operations and random disk writes. Cassandra uses an LSM-tree architecture where writes are appended to a commit log and stored in memtables before being flushed to immutable SSTables, optimizing for sequential disk IO and
