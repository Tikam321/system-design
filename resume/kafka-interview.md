x# Kafka Interview Questions & Answers

A structured set of Apache Kafka interview questions covering fundamentals, architecture, producers/consumers, partitions & replication, delivery semantics, Kafka Streams/Connect, performance tuning, and scenario-based questions.

---

## 1. Kafka Fundamentals

**Q1. What is Apache Kafka?**
Kafka is a distributed event streaming platform used for building real-time data pipelines and streaming applications. It's designed for high-throughput, fault-tolerant, publish-subscribe messaging, and can also durably store streams of records (acting as a distributed commit log).

**Q2. What are the main use cases of Kafka?**
- Real-time data pipelines between systems (ETL/CDC)
- Event-driven microservices communication
- Log aggregation
- Stream processing (real-time analytics, fraud detection)
- Messaging system replacement (decoupling producers/consumers)
- Metrics/monitoring data collection

**Q3. What are the core components of Kafka's architecture?**
- **Producer**: Publishes (writes) records to topics.
- **Consumer**: Subscribes to (reads) records from topics.
- **Broker**: A Kafka server that stores data and serves client requests. A cluster consists of multiple brokers.
- **Topic**: A category/feed name to which records are published.
- **Partition**: A topic is split into partitions for parallelism and scalability.
- **ZooKeeper / KRaft**: Manages cluster metadata, broker coordination, and leader election (KRaft is replacing ZooKeeper in newer versions).

**Q4. What is a Topic in Kafka?**
A topic is a logical channel/category to which producers send records and from which consumers read records. Topics are split into partitions and can have multiple producers/consumers.

**Q5. What is a Partition and why does Kafka use partitions?**
A partition is an ordered, immutable sequence of records within a topic — essentially an append-only log. Partitions allow:
- **Parallelism**: Multiple consumers can read different partitions simultaneously.
- **Scalability**: Partitions can be spread across multiple brokers.
- Each record within a partition gets a sequential ID called an **offset**.

**Q6. What is an Offset?**
A unique, sequential ID assigned to each record within a partition, representing its position in that partition's log. Consumers track offsets to know which records they've already read.

**Q7. Is the order of messages guaranteed in Kafka?**
Order is guaranteed **only within a single partition**, not across an entire topic. If ordering matters for related records (e.g., same customer ID), they should be sent to the same partition (typically by using the same key).

---

## 2. Brokers, Replication & Fault Tolerance

**Q8. What is a Kafka Broker?**
A broker is a single Kafka server that stores data (partitions) and handles read/write requests from producers and consumers. A Kafka cluster is made up of multiple brokers working together.

**Q9. What is replication in Kafka and why is it needed?**
Each partition can be replicated across multiple brokers (defined by the `replication.factor`) for fault tolerance. If a broker goes down, another broker holding a replica can take over, preventing data loss and downtime.

**Q10. What are Leader and Follower replicas?**
- **Leader**: The replica that handles all reads and writes for a partition.
- **Follower**: Replicas that passively replicate data from the leader. If the leader fails, one follower (that is in-sync) is elected as the new leader.

**Q11. What is an ISR (In-Sync Replica)?**
The set of replicas that are fully caught up with the leader's log (within an allowed lag). Only ISR members are eligible for leader election, ensuring no data loss during failover. Controlled by `replica.lag.time.max.ms`.

**Q12. What happens when a broker fails in Kafka?**
The controller detects the failure (via ZooKeeper/KRaft), and for every partition where the failed broker was the leader, a new leader is elected from the ISR. Producers/consumers reconnect to the new leader; clients automatically discover the new leader via metadata refresh.

**Q13. What is the role of the Controller broker?**
One broker in the cluster acts as the **Controller**, responsible for managing partition leader elections and broker state changes (join/leave cluster). Only one controller is active at a time.

**Q14. ZooKeeper vs KRaft — what's the difference?**
- **ZooKeeper**: Historically used by Kafka to store cluster metadata, manage broker membership, and perform leader election. Requires running a separate ZooKeeper ensemble.
- **KRaft (Kafka Raft)**: Kafka's newer built-in consensus protocol (based on Raft) that removes the ZooKeeper dependency entirely, simplifying deployment and improving scalability/metadata performance. KRaft is now the default and recommended mode in modern Kafka versions.

---

## 3. Producers

**Q15. What is a Kafka Producer?**
A client application that publishes (writes) records to one or more Kafka topics. Producers decide which partition a record goes to (via a partitioner, often based on the record's key).

**Q16. How does a Producer decide which partition to send a record to?**
- If a **key** is provided: a hash of the key determines the partition (ensuring records with the same key always go to the same partition, preserving order per key).
- If no key is provided: records are distributed in a round-robin (or sticky partitioner, in newer versions) fashion across partitions.
- A **custom partitioner** can also be implemented.

**Q17. What is the `acks` configuration in a Producer?**
Controls how many broker acknowledgments the producer waits for before considering a write successful:
- `acks=0`: No acknowledgment — fire and forget (fastest, risk of data loss).
- `acks=1`: Waits for leader acknowledgment only (moderate durability).
- `acks=all` (or `-1`): Waits for acknowledgment from all in-sync replicas (highest durability, slower).

**Q18. What is idempotent producer in Kafka?**
An idempotent producer (`enable.idempotence=true`) ensures that even if a producer retries sending a message (e.g., due to a network timeout), the message is **not duplicated** in the partition. Kafka achieves this using producer IDs and sequence numbers per partition.

**Q19. What is a Producer's retry mechanism?**
If a send fails (e.g., due to a leader election or transient network error), the producer will automatically retry based on `retries` and `retry.backoff.ms` settings. Combined with idempotence, this avoids duplicate messages even after retries.

**Q20. What is batching in Kafka producers, and why does it matter?**
Producers can batch multiple records destined for the same partition into a single request, controlled by `batch.size` and `linger.ms`. This improves throughput by reducing the number of network round-trips, at the cost of slightly higher latency per message.

---

## 4. Consumers & Consumer Groups

**Q21. What is a Kafka Consumer?**
A client application that reads (subscribes to) records from one or more Kafka topics/partitions.

**Q22. What is a Consumer Group?**
A group of consumers that work together to consume data from a topic, where each partition is consumed by exactly **one consumer within the group** at a time. This enables horizontal scaling of consumption — adding more consumers to a group increases parallelism (up to the number of partitions).

**Q23. What happens if there are more consumers than partitions in a group?**
Extra consumers beyond the number of partitions will sit **idle** — since each partition can only be assigned to one consumer within a group at a time, having more consumers than partitions doesn't increase parallelism.

**Q24. What is Consumer Rebalancing?**
When a consumer joins or leaves a group (or a partition is added), Kafka redistributes partition ownership among the remaining/new consumers in the group. This is called a rebalance and can temporarily pause consumption while it happens.

**Q25. What is the difference between `earliest` and `latest` in `auto.offset.reset`?**
- **earliest**: If no committed offset exists, start reading from the beginning of the partition.
- **latest**: If no committed offset exists, start reading only new messages produced after the consumer connects.

**Q26. What is offset commit, and what are the strategies for committing offsets?**
Committing an offset marks that a consumer has successfully processed records up to that point.
- **Automatic commit** (`enable.auto.commit=true`): Offsets committed periodically in the background — simpler but risks reprocessing or data loss on crash.
- **Manual commit** (`commitSync()` / `commitAsync()`): Application explicitly commits after processing, giving more control over "at-least-once" vs "at-most-once" semantics.

**Q27. What is the difference between `commitSync()` and `commitAsync()`?**
- **commitSync()**: Blocks until the broker confirms the commit — safer but slower, and will retry on failure.
- **commitAsync()**: Non-blocking, higher throughput, but doesn't automatically retry on failure (since retrying async commits out of order could commit an old offset over a newer one).

---

## 5. Delivery Semantics

**Q28. What delivery semantics does Kafka support?**
- **At-most-once**: Messages may be lost but are never redelivered (commit offset before processing).
- **At-least-once**: Messages are never lost but may be redelivered/duplicated (commit offset after processing) — the most common default.
- **Exactly-once**: Each message is processed exactly one time, no duplicates or loss — achieved using idempotent producers + transactions.

**Q29. How does Kafka achieve Exactly-Once Semantics (EOS)?**
Through a combination of:
- **Idempotent producers** (avoid duplicate writes on retry)
- **Transactional API** (`transactional.id`, `initTransactions()`, `commitTransaction()`) — allows atomic writes across multiple partitions/topics.
- **Read Process Write pattern** in Kafka Streams, using `isolation.level=read_committed` on the consumer side so it only reads committed transactional messages.

**Q30. What is the Transactional Producer used for?**
It allows a producer to write to multiple partitions/topics as a single atomic unit — either all writes succeed and are visible, or none are (if the transaction aborts). Commonly used in "consume-transform-produce" pipelines to guarantee exactly-once processing.

---

## 6. Kafka Streams & Kafka Connect

**Q31. What is Kafka Streams?**
A client library for building stream processing applications directly on top of Kafka — enabling transformations, aggregations, joins, and windowed computations on data streams, without needing a separate processing cluster (like Spark or Flink).

**Q32. What is a KStream vs a KTable?**
- **KStream**: Represents an unbounded stream of records — each record is an independent event (insert-only, like a log).
- **KTable**: Represents a changelog/table view of a stream — each new record with the same key **updates** the previous value (like a database table's current state).

**Q33. What is Kafka Connect?**
A framework for integrating Kafka with external systems (databases, file systems, cloud storage) using reusable, configuration-driven **connectors** — without writing custom producer/consumer code.
- **Source Connectors**: Pull data from an external system into Kafka.
- **Sink Connectors**: Push data from Kafka to an external system.

**Q34. What is a Kafka Connector vs a Kafka Task?**
- **Connector**: Defines the configuration and coordinates the work (e.g., which database/table to read).
- **Task**: The actual unit of work that performs the data copying; a connector can be split into multiple parallel tasks for scalability.

---

## 7. Performance, Retention & Storage

**Q35. How does Kafka achieve high throughput?**
- Sequential disk I/O (append-only log) instead of random access
- Zero-copy data transfer (`sendfile()` system call, avoiding unnecessary data copies)
- Batching and compression of messages
- Partitioning for parallelism across brokers/consumers
- Efficient binary protocol over TCP

**Q36. What is log compaction in Kafka?**
A retention strategy where Kafka retains only the **latest value for each key** in a partition (instead of deleting based on time/size), removing older duplicate-key records. Useful for maintaining a "current state" changelog (e.g., for KTables or restoring application state).

**Q37. What is the difference between log retention by time vs by size?**
- **Time-based** (`log.retention.hours`): Deletes records older than a configured time period (default 168 hours/7 days).
- **Size-based** (`log.retention.bytes`): Deletes the oldest records once the partition log exceeds a configured size.
Both can be configured together; whichever limit is hit first triggers deletion.

**Q38. What is message compression in Kafka, and what types are supported?**
Producers can compress batches of messages to reduce network and storage usage. Supported codecs: `gzip`, `snappy`, `lz4`, and `zstd`. Compression happens on the producer side and is decompressed by the consumer (brokers store and forward compressed batches as-is for efficiency).

**Q39. What is the purpose of `min.insync.replicas`?**
Works together with `acks=all` — it specifies the minimum number of in-sync replicas that must acknowledge a write for it to be considered successful. If fewer than this number of replicas are in sync, the producer will receive an error, protecting against data loss when replicas are down.

---

## 8. Monitoring & Operations

**Q40. How do you monitor Kafka in production?**
- **JMX metrics**: Broker, producer, and consumer metrics (throughput, latency, under-replicated partitions).
- **Consumer lag monitoring**: Tools like Burrow, Kafka's own `kafka-consumer-groups.sh`, or Confluent Control Center to track how far behind consumers are.
- **Third-party tools**: Prometheus + Grafana (via JMX Exporter), Datadog, Confluent Control Center, LinkedIn Cruise Control (for partition rebalancing/optimization).

**Q41. What is Consumer Lag and why does it matter?**
The difference between the latest offset produced to a partition and the offset a consumer has committed/processed. High or growing lag indicates the consumer isn't keeping up with the rate of incoming data — a key metric for detecting performance bottlenecks.

**Q42. How do you scale a Kafka cluster?**
- Add more brokers to the cluster and let partition reassignment redistribute load (via `kafka-reassign-partitions.sh` or tools like Cruise Control).
- Increase the number of partitions per topic to allow more consumer parallelism (note: partitions can only be increased, not decreased, on an existing topic).
- Tune producer/consumer configs (batch size, fetch size, compression) for throughput.

**Q43. What happens if you increase the number of partitions on an existing topic?**
New partitions are added, but existing message-to-partition mappings for **keyed messages don't change retroactively** — meaning the same key could now hash to a different partition than before, potentially breaking ordering guarantees for that key going forward. Partition counts should ideally be planned upfront.

---

## 9. Scenario-Based Questions

**Q44. How would you design a Kafka-based system to process messages exactly once end-to-end?**
- Use an **idempotent producer** with `acks=all` to avoid duplicate writes.
- Use the **transactional API** for consume-transform-produce workflows spanning multiple topics.
- Set consumer `isolation.level=read_committed` so it only reads committed transactional records.
- Make downstream processing idempotent as well (e.g., upserts keyed by a unique ID) as a safety net, since true end-to-end exactly-once also depends on the sink system's behavior.

**Q45. A consumer group is falling behind (high lag) during peak traffic. How would you fix it?**
- Increase the number of partitions (and correspondingly, consumers) to increase parallelism.
- Optimize the consumer's processing logic (reduce per-record processing time, batch downstream writes).
- Tune consumer configs like `fetch.min.bytes`, `max.poll.records`, `max.poll.interval.ms`.
- Scale out horizontally by adding more consumer instances (up to the partition count).
- Check for uneven partition distribution (data skew) if a few partitions have much more traffic.

**Q46. How would you ensure message ordering for a specific entity (e.g., all events for a given user)?**
Use the entity's ID (e.g., `user_id`) as the **message key**. Kafka's default partitioner hashes the key to consistently route all messages with that key to the same partition, and order is guaranteed within a partition.

**Q47. How would you handle a "poison pill" message that repeatedly crashes a consumer?**
- Implement error handling/try-catch around message processing so a bad message doesn't block the consumer indefinitely.
- Route failed messages to a **Dead Letter Queue (DLQ)** topic for later inspection/reprocessing.
- Use retry logic with a limited number of attempts before moving to the DLQ.
- Alert/monitor DLQ topic size so issues are caught quickly.

**Q48. How would you migrate a Kafka cluster from ZooKeeper mode to KRaft mode?**
Kafka provides a documented migration path: enable dual-write/migration mode where KRaft controllers run alongside ZooKeeper temporarily, migrate broker metadata over, then fully cut over to KRaft controllers once migration completes — done gradually to avoid downtime. (Always check the specific Kafka version's official migration guide, as steps evolve between releases.)

**Q49. How is Kafka different from traditional message queues like RabbitMQ?**
| Aspect | Kafka | Traditional MQ (e.g., RabbitMQ) |
|---|---|---|
| Model | Distributed log, pub-sub with retention | Queue-based, often push-based |
| Message retention | Retains messages for a configured period (or forever), even after consumption | Typically deleted once consumed/acknowledged |
| Throughput | Very high (built for large-scale streaming) | Generally lower, optimized for complex routing |
| Replay | Consumers can re-read old messages (by resetting offsets) | Usually not supported once consumed |
| Ordering | Guaranteed per partition | Depends on queue configuration |
| Use case | Event streaming, log aggregation, big data pipelines | Task queues, RPC, complex routing (e.g., with exchanges) |

**Q50. What is the difference between Kafka and a database?**
Kafka is primarily an **event streaming/log system** optimized for high-throughput sequential writes and real-time consumption by multiple subscribers, whereas a database is optimized for structured storage, complex queries, transactions, and random access/updates to current-state data. Kafka can complement a database (e.g., for CDC, event sourcing) rather than replace it, though features like log compaction give it some database-like properties for key-based current-state data.

---

## Tips for the Interview
- Be very clear on the difference between **partition-level ordering** vs **topic-level ordering** — this trips up a lot of candidates.
- Know **acks, idempotence, and transactions** cold — exactly-once semantics is one of the most commonly probed deep-dive areas.
- Understand **consumer group rebalancing** and its operational impact (brief unavailability during rebalance).
- Be ready to explain **real-world architecture** you've built or worked with (producers, topics, partition strategy, consumer scaling, error handling/DLQ).
- Brush up on **KRaft vs ZooKeeper**, since many teams are migrating and interviewers may probe current-state knowledge here.
