# Kafka Interview Questions & Answers

A collection of commonly asked Apache Kafka interview questions with clear, interview-ready answers — covering core concepts, architecture, producer/consumer internals, and integration with Java and Spring Boot.

---

## Core Concepts

**1. What is Apache Kafka?**
Kafka is a distributed event streaming platform used to publish, subscribe to, store, and process streams of records in real time. It's built to handle high-throughput, fault-tolerant, and durable messaging between systems. Unlike traditional message queues, Kafka retains messages for a configurable period, so multiple consumers can read the same data independently. It's commonly used for building real-time data pipelines, event-driven microservices, and stream processing applications.

**2. What are the main components of Kafka's architecture?**
Kafka's architecture consists of Producers (which publish messages), Consumers (which read messages), Brokers (servers that store and serve data), Topics (logical channels for messages), and Partitions (the unit of parallelism within a topic). Kafka also relies on a metadata layer — historically ZooKeeper, and now KRaft (Kafka Raft) in newer versions — to manage broker coordination, leader election, and cluster metadata.

**3. What is a Topic in Kafka?**
A topic is a named stream of records that producers write to and consumers read from. It acts like a category or feed name. Topics are split into partitions so that data can be distributed across multiple brokers, enabling parallel reads and writes. Each message within a topic is immutable once written.

**4. What is a Partition, and why does Kafka use them?**
A partition is an ordered, immutable sequence of records within a topic, and it's the basic unit of parallelism in Kafka. Partitions allow a topic's data to be split across multiple brokers so that reads and writes can scale horizontally. Order is guaranteed only within a single partition, not across the entire topic, which is an important detail when designing systems that depend on message ordering.

**5. What is an Offset in Kafka?**
An offset is a unique, sequential ID assigned to each record within a partition, marking its position in that partition's log. Consumers use offsets to track which messages they've already processed, so they know where to resume reading after a restart or rebalance. Offsets are partition-specific, meaning the same offset number in two different partitions refers to two completely different messages.

**6. What is a Consumer Group?**
A consumer group is a set of consumers that work together to read data from a topic, with each partition assigned to only one consumer within the group at a time. This allows Kafka to parallelize message consumption across multiple consumer instances while ensuring each message is processed once per group. Different consumer groups reading the same topic are independent of each other, which is what enables Kafka's publish-subscribe model.

**7. What is Replication in Kafka, and why is it important?**
Replication means each partition's data is copied across multiple brokers, with one broker acting as the leader and the others as followers. If the leader broker fails, one of the in-sync followers is promoted to leader, so the system stays available without data loss. The replication factor is a key configuration for balancing durability against storage and network overhead.

**8. What is an In-Sync Replica (ISR)?**
An ISR is a replica that has fully caught up with the leader's log within an allowed lag threshold. Only replicas in this set are eligible to be elected as the new leader if the current leader fails, which protects against data loss. Kafka tracks the ISR list dynamically — a replica falls out of the ISR if it lags too far behind, and can rejoin once it catches up.

---

## Producers and Consumers

**9. How does a Kafka Producer decide which partition to send a message to?**
If a message has a key, Kafka uses a hash of the key to consistently route it to the same partition, which preserves ordering for that key. If there's no key, Kafka uses a partitioner (round-robin or sticky partitioning in newer versions) to spread messages evenly across partitions. Producers can also implement a custom partitioner if they need specific routing logic.

**10. What is the difference between `acks=0`, `acks=1`, and `acks=all`?**
`acks=0` means the producer doesn't wait for any acknowledgment, giving the highest throughput but the weakest durability guarantee. `acks=1` means the producer waits for the leader broker to acknowledge the write, which balances speed and safety but risks loss if the leader fails before replication. `acks=all` (or `-1`) means the producer waits for all in-sync replicas to acknowledge, giving the strongest durability guarantee at the cost of higher latency.

**11. What is Consumer Lag, and why does it matter?**
Consumer lag is the difference between the latest offset produced to a partition and the offset a consumer has last committed. High lag indicates the consumer isn't keeping up with incoming data, which could point to slow processing logic, insufficient consumer instances, or resource bottlenecks. Monitoring lag is essential for spotting performance issues before they cause noticeable delays downstream.

**12. What happens during a Consumer Rebalance?**
A rebalance occurs when partitions are reassigned among the consumers in a group — for example, when a consumer joins, leaves, or crashes. During a rebalance, consumption pauses briefly while the group coordinator redistributes partitions, which can cause temporary processing delays. Kafka has introduced cooperative rebalancing strategies to reduce the "stop-the-world" effect of older eager rebalancing protocols.

**13. What's the difference between at-most-once, at-least-once, and exactly-once delivery semantics?**
At-most-once means a message might be lost but never processed twice, typically because the offset is committed before processing completes. At-least-once means a message is never lost but might be processed more than once, usually because the offset is committed after processing. Exactly-once means each message is processed once and only once, which Kafka achieves through idempotent producers and transactional APIs working together with consumer offset commits.

**14. How does Kafka achieve idempotent producers?**
Kafka assigns each producer a unique Producer ID (PID) and tags each message with a sequence number per partition. The broker uses this PID and sequence number to detect and discard duplicate writes caused by retries, ensuring a message is written only once even if the producer resends it due to a network timeout. This is enabled with `enable.idempotence=true` and is a foundation for Kafka's exactly-once semantics.

---

## Storage and Reliability

**15. How does Kafka ensure durability of messages?**
Kafka persists every message to disk as part of the partition log before considering it committed, rather than keeping data only in memory. Combined with replication across multiple brokers, this means data survives both broker restarts and broker failures. Producers can further control durability guarantees using the `acks` setting depending on how critical it is that no data is lost.

**16. What is Log Compaction in Kafka?**
Log compaction is a retention strategy where Kafka keeps only the latest value for each unique key in a topic, rather than deleting messages purely based on age. This is useful for use cases like maintaining the latest state of an entity — for example, a changelog of user profile updates — where only the most recent record per key matters. It contrasts with the default time- or size-based retention policy that deletes old records regardless of key.

**17. What is the role of ZooKeeper in Kafka, and how does KRaft change this?**
Historically, ZooKeeper managed cluster metadata, broker registration, and controller election for Kafka. KRaft (Kafka Raft) removes this dependency by having Kafka brokers themselves manage metadata using the Raft consensus protocol, which simplifies deployment and improves scalability. Newer Kafka versions are moving toward KRaft as the default, phasing out ZooKeeper entirely.

**18. What is retention in Kafka, and how is it configured?**
Retention determines how long Kafka keeps messages in a topic before deleting them, controlled by settings like `retention.ms` (time-based) or `retention.bytes` (size-based). This is independent of whether consumers have read the messages — Kafka doesn't delete data just because it's been consumed. This design lets multiple consumers, including ones added later, read historical data within the retention window.

---

## Advanced Topics

**19. What is Kafka Streams?**
Kafka Streams is a Java library for building stream processing applications directly on top of Kafka, without needing a separate processing cluster like Spark or Flink. It allows operations like filtering, joining, aggregating, and windowing data in real time, and it reads from and writes back to Kafka topics. It also handles state management and fault tolerance internally using changelog topics.

**20. What is Kafka Connect?**
Kafka Connect is a framework for integrating Kafka with external systems like databases, file systems, or cloud storage, using reusable source and sink connectors. Instead of writing custom producer or consumer code for every integration, teams can configure existing connectors (like JDBC or Debezium) to move data in and out of Kafka. It supports distributed and standalone modes and handles offset tracking and fault tolerance automatically.

**21. What is a Transactional Producer in Kafka?**
A transactional producer allows a set of writes to multiple partitions (and even multiple topics) to be committed atomically, so consumers only see the complete set of writes or none at all. This is essential for exactly-once processing, especially in read-process-write patterns like Kafka Streams. It's configured using a `transactional.id` and requires the producer to explicitly begin, commit, or abort transactions.

**22. How does Kafka handle Leader Election?**
When a partition's leader broker fails, the controller broker selects a new leader from the partition's in-sync replicas. This process is coordinated through the controller, which is elected among the brokers themselves (via ZooKeeper previously, or via the KRaft quorum now). Choosing a leader only from the ISR ensures no committed data is lost during the failover.

**23. What is the difference between Kafka and traditional message queues like RabbitMQ?**
Traditional queues like RabbitMQ typically delete a message once it's consumed and acknowledged, making it a transient messaging system. Kafka instead retains messages for a configured retention period regardless of consumption, allowing multiple independent consumers to replay and reprocess the same data. Kafka is also optimized for high-throughput, ordered, partitioned logs, whereas RabbitMQ is optimized for flexible routing and lower-latency point-to-point or fan-out messaging.

---

## Kafka with Java

**24. How do you create a simple Kafka Producer in Java?**
You configure a `Properties` object with settings like `bootstrap.servers`, `key.serializer`, and `value.serializer`, then pass it to a `KafkaProducer` instance. Messages are sent using `producer.send()`, wrapped in a `ProducerRecord` that specifies the topic, optional key, and value. It's good practice to call `producer.flush()` and `producer.close()` to ensure all messages are delivered before the application exits.

**25. How do you create a simple Kafka Consumer in Java?**
You configure a `Properties` object with `bootstrap.servers`, `group.id`, `key.deserializer`, and `value.deserializer`, then create a `KafkaConsumer` and subscribe it to one or more topics. In a polling loop, you call `consumer.poll()` to fetch records, process them, and either rely on auto-commit or manually commit offsets using `commitSync()` or `commitAsync()`. Manual commits give more control over exactly when a message is considered "processed."

**26. What's the difference between `commitSync()` and `commitAsync()` in the Java Consumer API?**
`commitSync()` blocks until the broker acknowledges the offset commit, making it more reliable but slower since it can retry on failure. `commitAsync()` sends the commit request without waiting for a response, which improves throughput but means failures aren't automatically retried in the same way. A common pattern is to use `commitAsync()` for regular commits during processing and a final `commitSync()` before shutting down the consumer, to guarantee the last offsets are saved.

**27. How do you handle serialization of custom Java objects in Kafka?**
You implement the `Serializer<T>` and `Deserializer<T>` interfaces (or use a library like Jackson for JSON, or Avro/Protobuf with a schema registry) to convert your custom objects to and from byte arrays. These are then configured as the `key.serializer`/`value.serializer` and their deserializer counterparts in producer and consumer properties. Using a schema-based format like Avro with a Schema Registry is preferred in production because it enforces contract compatibility between producers and consumers as schemas evolve.

---

## Kafka with Spring Boot

**28. How do you configure a Kafka Producer in Spring Boot?**
Spring Boot uses `spring-kafka` and exposes configuration through `application.yml` or `application.properties`, typically under the `spring.kafka.producer` prefix, for settings like bootstrap servers, key/value serializers, and acks. Spring Boot auto-configures a `KafkaTemplate` bean, which you can autowire directly into your service classes to send messages with `kafkaTemplate.send(topic, key, value)`. This removes the need to manually manage `KafkaProducer` lifecycle.

**29. How do you consume messages in Spring Boot using Kafka?**
You annotate a method with `@KafkaListener(topics = "topic-name", groupId = "group-id")`, and Spring Kafka handles subscribing, polling, and dispatching records to that method automatically. The method parameter type can be the raw message payload, or a `ConsumerRecord` if you need access to metadata like partition and offset. Spring also supports manual acknowledgment mode via `AckMode.MANUAL` when you need explicit control over offset commits.

**30. What is `KafkaTemplate` in Spring Kafka?**
`KafkaTemplate` is Spring's high-level abstraction over the native `KafkaProducer`, simplifying how messages are sent to Kafka topics. It handles the boilerplate of producer configuration and provides convenient methods like `send()` that return a `ListenableFuture` (or `CompletableFuture` in newer versions) so you can attach callbacks for success or failure. It integrates naturally with Spring's dependency injection, so you just autowire it wherever you need to publish messages.

**31. How does error handling work in Spring Kafka listeners?**
Spring Kafka provides `DefaultErrorHandler` (formerly `SeekToCurrentErrorHandler`), which lets you configure retry behavior and dead-letter topic routing when message processing fails. You can specify backoff policies for retries and, after exhausting retries, automatically publish the failed message to a dead-letter topic using `DeadLetterPublishingRecoverer`. This keeps a single bad message from blocking the entire partition while still preserving it for later inspection.

**32. What is `@KafkaListener`'s concurrency setting, and how does it relate to partitions?**
The `concurrency` attribute on `@KafkaListener` defines how many consumer threads Spring Kafka spins up for that listener, effectively creating multiple consumer instances within the same consumer group. Since Kafka only allows one consumer per partition within a group at a time, the useful concurrency is capped by the number of partitions in the topic — extra threads beyond that will sit idle. This setting is a straightforward way to scale message processing without manually managing threads.

**33. How do you test Kafka-based Spring Boot applications?**
Spring Kafka provides an embedded, in-memory Kafka broker (`@EmbeddedKafka`) for integration tests, allowing you to spin up a real broker instance without external infrastructure. You can then produce test messages, invoke listeners, and assert on the resulting behavior or state changes. For unit tests that don't need a real broker, you typically mock `KafkaTemplate` and verify interactions rather than testing actual message delivery.

---

## Practical / Scenario-Based

**34. How would you handle duplicate message processing in a consumer?**
The most reliable approach is to make consumer processing idempotent — for example, using a unique message ID or key to check whether it's already been processed before applying changes, often backed by a database unique constraint or an idempotency table. Combining this with Kafka's own idempotent producer and, where needed, transactional writes reduces duplicates at the source, but downstream idempotency is still the safety net since exactly-once guarantees don't extend past Kafka itself into external systems. This is especially important when consumers write to databases or call external APIs as part of processing.

**35. How do you handle a "poison pill" message that keeps crashing your consumer?**
A poison pill is a malformed or unprocessable message that causes repeated failures every time the consumer retries it, potentially blocking the entire partition. The typical fix is to configure an error handler with a bounded number of retries, after which the message is routed to a dead-letter topic instead of endlessly retried. This allows the consumer to move past the bad message and keep processing, while the problematic record is preserved separately for manual investigation.

**36. How would you scale a Kafka consumer application to handle higher throughput?**
The main lever is increasing the number of partitions for the topic, since parallelism in Kafka is bounded by the partition count within a consumer group. From there, you can add more consumer instances (up to the number of partitions) or increase per-consumer processing efficiency through batching, asynchronous processing, or tuning `max.poll.records` and `fetch.min.bytes`. It's worth noting that increasing partitions after a topic is created can affect key-based ordering, so partition counts are best planned with future scale in mind.

**37. What would you check if you noticed increasing consumer lag in production?**
First, check whether the consumers are actually running and healthy, since a crashed or stuck consumer instance is the most common cause. Next, look at processing time per message or batch, resource utilization (CPU, memory, GC pauses), and whether the workload has genuinely increased beyond current capacity. If the consumer logic itself is efficient, the fix is usually to add more partitions and consumer instances, or optimize slow downstream calls like database writes or external API requests happening inside the listener.

---

## ZooKeeper Interview Questions

**38. What is ZooKeeper, and why was it used with Kafka?**
ZooKeeper is a distributed coordination service that manages configuration, naming, synchronization, and group membership for distributed systems. In older Kafka versions, it served as the source of truth for cluster metadata — tracking which brokers are alive, storing topic configuration, and managing controller election. Kafka needed something like ZooKeeper because coordinating this kind of shared state reliably across multiple machines is a hard problem on its own, and ZooKeeper solved it in a reusable, battle-tested way.

**39. What are Znodes in ZooKeeper?**
A znode is the basic data unit in ZooKeeper's hierarchical namespace, similar to a file or directory in a filesystem, and each znode can hold a small amount of data along with metadata like version numbers and timestamps. Znodes can be persistent (remaining until explicitly deleted) or ephemeral (automatically deleted when the client session that created them ends). Kafka historically used ephemeral znodes to represent live broker registrations, so a broker's znode would disappear automatically if it crashed or lost connection.

**40. What is the difference between a Persistent and an Ephemeral Znode?**
A persistent znode stays in ZooKeeper's tree until a client explicitly deletes it, making it suitable for storing durable configuration or metadata. An ephemeral znode is tied to the client session that created it and is automatically removed the moment that session ends, whether due to a clean disconnect or a crash. This ephemeral behavior is what made ZooKeeper useful for liveness tracking in Kafka — a broker's presence in ZooKeeper directly reflected whether it was actually alive and connected.

**41. What is the role of the ZooKeeper Ensemble, and why does it use an odd number of nodes?**
A ZooKeeper ensemble is a cluster of ZooKeeper servers working together to provide fault tolerance and high availability for the coordination service itself. It uses an odd number of nodes (typically 3, 5, or 7) because ZooKeeper relies on majority voting (quorum) to agree on state changes, and an odd count avoids tie situations while maximizing fault tolerance for a given number of servers — for example, a 5-node ensemble can tolerate 2 failures and still maintain a majority.

**42. What is Leader Election in ZooKeeper, and how does it relate to Kafka's Controller?**
Within a ZooKeeper ensemble, one server is elected as the leader responsible for handling all write requests and coordinating updates to the other servers, while the rest act as followers serving read requests. Kafka historically used this same underlying mechanism to elect its own Controller broker — the broker responsible for managing partition leader elections and cluster-wide metadata changes. If the controller broker failed, ZooKeeper would detect the ephemeral znode disappearing and trigger a new controller election among the remaining brokers.

**43. What consistency guarantees does ZooKeeper provide?**
ZooKeeper guarantees sequential consistency, meaning updates from a client are applied in the order they were sent, and atomicity, meaning updates either fully succeed or fully fail with no partial state. It also guarantees that once a write is acknowledged, all clients will eventually observe it, with the leader always reflecting the latest state. These guarantees are what make it safe to use for coordination tasks like leader election and distributed locks, where partial or out-of-order updates could cause serious bugs.

**44. What is a ZooKeeper Watch?**
A watch is a one-time trigger a client sets on a znode to get notified when that znode changes — for example, when its data is updated or it's deleted. Kafka used watches extensively to react to cluster events, such as detecting when a new broker joined, an existing broker went down, or topic configuration changed, without having to constantly poll ZooKeeper for updates. Watches are single-fire by design, meaning the client has to re-register the watch after it triggers if it wants to keep observing future changes.

**45. What are the main drawbacks of relying on ZooKeeper in a Kafka deployment?**
Running ZooKeeper means operating and maintaining a second distributed system alongside Kafka itself, which adds operational complexity, extra infrastructure, and another potential point of failure to monitor. As cluster metadata grows — particularly the number of partitions — ZooKeeper can become a scalability bottleneck, since all metadata changes and watches funnel through it. These pain points were the main motivation behind Kafka's move to KRaft, which folds metadata management directly into Kafka brokers and removes the ZooKeeper dependency entirely.

**46. How does KRaft replace ZooKeeper in modern Kafka?**
KRaft (Kafka Raft) is Kafka's built-in consensus protocol that lets a subset of Kafka brokers, called controllers, manage cluster metadata directly using the Raft algorithm instead of delegating it to an external ZooKeeper ensemble. Metadata is stored as a replicated log (the metadata log) within Kafka itself, replicated across controller nodes for fault tolerance, the same way partition data is replicated across brokers. This simplifies deployment to a single system, improves metadata scalability for clusters with very large numbers of partitions, and is now the recommended mode for new Kafka clusters.

**47. If you were troubleshooting a ZooKeeper-based Kafka cluster and saw frequent leader elections, what would you investigate?**
Frequent, unexpected leader elections often point to instability in ZooKeeper sessions — commonly caused by network latency or partitioning, long garbage collection pauses on brokers causing session timeouts, or an under-resourced ZooKeeper ensemble struggling to keep up with the volume of metadata updates. I'd check ZooKeeper's own logs and metrics for session expirations, review broker GC logs for long pauses, and confirm that session timeout and heartbeat settings are appropriately tuned for the environment's network conditions.

---
