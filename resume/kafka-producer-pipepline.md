# Kafka Producer Pipeline — Interview Preparation Guide

> **How to explain:** "I built a producer-side event-driven pipeline for a chat messaging service using Spring Boot and Kafka, achieving 30K events/sec throughput with zero data loss."

---

## 1. Project Overview (30-Second Elevator Pitch)

```
┌─────────────────────────────────────────────────────────────┐
│  WHAT: Event-driven pipeline for chat messages              │
│  HOW:  Spring Boot 3.5.3 + Kafka 3.x (Confluent 7.6.0)     │
│  WHY:  Decouple message ingestion from processing           │
│  RESULT: 30K events/sec, < 100ms p95 latency, 0% failures  │
└─────────────────────────────────────────────────────────────┘
```

### One-Liner for Interview

> "I implemented a Kafka producer-side event pipeline that ingests chat messages via REST API and asynchronously publishes them to Kafka, achieving 30K events/sec with exactly-once semantics and zero data loss."

---

## 2. Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    SYSTEM ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐             │
│  │  Client   │───►│  REST    │───►│ Service  │             │
│  │  (HTTP)   │    │Controller│    │  Layer   │             │
│  └──────────┘    └──────────┘    └─────┬────┘             │
│                                        │                    │
│                                        ▼                    │
│                              ┌──────────────────┐          │
│                              │  KafkaTemplate   │          │
│                              │  (Async Send)    │          │
│                              └────────┬─────────┘          │
│                                       │                     │
│                                       ▼                     │
│                              ┌──────────────────┐          │
│                              │  Producer Buffer │          │
│                              │  (32MB)          │          │
│                              └────────┬─────────┘          │
│                                       │                     │
│                                       ▼                     │
│                              ┌──────────────────┐          │
│                              │  Sender Thread   │          │
│                              │  (Batching)      │          │
│                              └────────┬─────────┘          │
│                                       │                     │
│                                       ▼                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              KAFKA BROKER CLUSTER                    │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐             │   │
│  │  │ Broker 1│  │ Broker 2│  │ Broker 3│             │   │
│  │  │ P0,P3,  │  │ P1,P4,  │  │ P2,P5,  │             │   │
│  │  │ P6,P9   │  │ P7,P10  │  │ P8,P11  │             │   │
│  │  └─────────┘  └─────────┘  └─────────┘             │   │
│  │                                                     │   │
│  │  Topic: chat-messages (12 partitions, RF=3)         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Key Configuration (What to Say in Interview)

### Producer Configuration

| Parameter | Value | Interview Explanation |
|-----------|-------|----------------------|
| `acks` | `all` | "All replicas must acknowledge before considering write successful. Ensures zero data loss even if a broker dies." |
| `batch.size` | 32KB | "Groups multiple messages into one network request. Reduces overhead of individual sends." |
| `linger.ms` | 10ms | "Waits up to 10ms to fill batch before sending. Trades 10ms latency for better throughput." |
| `compression.type` | lz4 | "Best balance of compression ratio and CPU cost. Reduces network I/O by ~60%." |
| `buffer.memory` | 32MB | "Async send buffer. Messages queue here if sender thread is busy. Prevents blocking." |
| `enable.idempotence` | true | "Exactly-once semantics. Deduplicates messages if retries happen." |
| `max.in.flight.requests` | 5 | "Allows 5 concurrent batches in flight. Improves throughput without ordering issues." |
| `retries` | MAX_INT | "Retry indefinitely on transient failures." |

### Why These Values?

```
Interviewer: "Why acks=all instead of acks=1?"

Answer:
  "acks=1 means only the leader acknowledges — if leader dies before
   replicating, data is lost. acks=all ensures all ISR replicas have
   the message before we consider it sent. For a chat system, losing
   messages is unacceptable."

Interviewer: "Why linger.ms=10 instead of 0?"

Answer:
  "linger.ms=0 means send immediately — creates many small batches
   (1 message per batch). linger.ms=10 waits 10ms to fill the batch.
   At high throughput, this means 100x fewer network calls. The trade-off
   is 10ms latency, which is acceptable for chat."

Interviewer: "Why lz4 compression?"

Answer:
  "lz4 has the best speed-to-compression ratio. gzip compresses more
   but uses too much CPU. lz4 reduces network I/O by ~60% with minimal
   CPU overhead. zstd is newer but lz4 is more battle-tested."
```

---

## 4. Topic Design

### Partition Strategy

| Aspect | Value | Reasoning |
|--------|-------|-----------|
| Partitions | 12 | Fixed by consumer constraint (not our choice) |
| Replication Factor | 3 | One per broker, survives 2 broker failures |
| min.insync.replicas | 2 | At least 2 replicas must have data before ack |
| Retention | 7 days | Standard for chat messages |
| Cleanup | delete | Old messages removed after retention |

### Why 12 Partitions?

```
Interviewer: "Why 12 partitions?"

Answer:
  "The consumer team has a constraint of 12 partitions max.
   We designed the system to maximize throughput within this limit.
   Each partition handles ~2,500 events/sec, giving us 30K total.
   If the constraint lifts, we can scale to 24+ partitions for 60K+."
```

### Partition Distribution

```
Partition 0  → Broker 1 (NVMe SSD)
Partition 1  → Broker 2 (NVMe SSD)
Partition 2  → Broker 3 (NVMe SSD)
Partition 3  → Broker 1
Partition 4  → Broker 2
Partition 5  → Broker 3
...
Partition 11 → Broker 3

Even distribution across 3 brokers = balanced I/O
```

---

## 5. Throughput Analysis (The Numbers)

### Benchmarking Results

| Metric | Value | Context |
|--------|-------|---------|
| Max Throughput | **30,000 events/sec** | Stable, 0% failures |
| Per Partition | **~2,500 events/sec** | 30K ÷ 12 partitions |
| p50 Latency | ~40ms | At ceiling |
| p95 Latency | ~100ms | At ceiling |
| p99 Latency | ~150ms | At ceiling |
| Failure Rate | 0% | At ceiling |
| At 40K | 4.8% failures, 789ms p99 | Breaking point |

### How We Found the Ceiling

```
Interviewer: "How did you determine the maximum throughput?"

Answer:
  "We used k6 with a ramping-arrival-rate test — starting at 1K
   requests/sec and increasing by 5K every minute until failures
   appeared. We found the system handles 30K events/sec with zero
   failures, but at 40K we see 4.8% error rate and p99 spikes to
   789ms. So our ceiling is 30K."
```

### Why 30K is the Ceiling

```
Interviewer: "What's the bottleneck at 30K?"

Answer:
  "The single Kafka sender thread. Even though we have 250 HTTP
   threads calling KafkaTemplate.send(), the internal sender thread
   that batches and sends to Kafka can only process ~30K messages/sec.
   The buffer fills up, and beyond that we get BufferExhaustedException."
```

---

## 6. Data Flow Walkthrough

### Step-by-Step (For Interview)

```
1. HTTP POST arrives at /api/v1/messages
   └─► MessageController receives request

2. Controller validates and calls MessageService
   └─► Service creates MessageDto record

3. Service calls KafkaTemplate.send()
   └─► Message goes to producer buffer (async, non-blocking)
   └─► Returns immediately with ListenableFuture

4. Sender Thread picks up message from buffer
   └─► Waits up to 10ms (linger.ms) to fill batch
   └─► Compresses batch with lz4
   └─► Sends batch to Kafka leader broker

5. Kafka Broker writes to disk
   └─► Replicates to 2 other brokers (RF=3)
   └─► All 3 replicas acknowledge (acks=all)

6. Producer receives acknowledgment
   └─► Callback fires: onSuccess or onFailure
   └─► Metrics updated (Micrometer)
   └─► Failed messages logged for retry
```

### Request Timeline

```
Time 0ms:     HTTP request arrives
Time 1ms:     Controller validates, calls service
Time 2ms:     KafkaTemplate.send() — message in buffer
Time 2ms:     Response sent to client (2ms total!)
Time 12ms:    Sender thread picks up message (after linger.ms)
Time 13ms:    Batch sent to Kafka
Time 15ms:    Kafka acks (all replicas)
              Total time: 15ms (client sees 2ms)
```

---

## 7. Interview Questions & Answers

### Architecture Questions

**Q: "How does Kafka achieve high throughput?"**

> "Kafka uses sequential I/O, zero-copy transfer, and batching. Instead of writing each message individually, it batches thousands of messages into a single network call. This reduces syscall overhead and leverages OS page cache."

**Q: "What is the difference between at-least-once, at-most-once, and exactly-once?"**

> - **At-most-once:** Send and forget. If broker dies, message is lost.
> - **At-least-once:** Retry on failure. May send duplicates.
> - **Exactly-once:** Idempotent producer + transactional consumer. No duplicates, no loss.
>
> "We use exactly-once via `enable.idempotence=true`. Kafka assigns each batch a sequence number, so duplicates are automatically rejected."

**Q: "Why async send instead of sync?"**

> "Sync send blocks until Kafka acks — at 30K/sec, each request would wait 10-15ms. Async returns immediately, letting the HTTP layer handle 30K+ concurrent requests. The sender thread handles batching in the background."

**Q: "What happens if Kafka broker goes down?"**

> "With replication factor 3 and min.insync.replicas=2, the system continues if 1 broker dies. The remaining 2 replicas have the data. If 2 brokers die, writes fail (we can't guarantee durability)."

---

### Configuration Questions

**Q: "Why batch.size=32KB?"**

> "It's a balance between latency and throughput. Larger batches (256KB) improve throughput but increase latency. 32KB gives us sub-100ms p95 while still batching effectively. We can tune to 128KB if we need 50K+ throughput."

**Q: "What does acks=all mean?"**

> "The leader broker writes the message to its log, then replicates to all ISR (in-sync replicas) brokers. Only when ALL replicas acknowledge does the producer consider the send successful. This guarantees no data loss even if the leader dies immediately after write."

**Q: "How does lz4 compression help?"**

> "lz4 reduces the size of each batch by ~60%. At 30K events/sec, that's ~30MB/sec of data. With lz4, we only send ~12MB/sec over the network. This reduces network I/O and allows the sender thread to process more messages per second."

---

### Throughput Questions

**Q: "How did you achieve 30K events/sec?"**

> "Three key optimizations:
> 1. **Batching:** 32KB batches with 10ms linger — groups messages efficiently
> 2. **Async send:** Non-blocking KafkaTemplate.send() — HTTP layer doesn't wait
> 3. **Compression:** lz4 — reduces network I/O by 60%
>
> Without these, we'd be limited to ~5K events/sec."

**Q: "How would you scale to 100K events/sec?"**

> "Four approaches, in order of effectiveness:
> 1. **Tune batching:** Increase batch.size to 256KB, linger.ms to 50ms → 50K
> 2. **Add producer instances:** 4 KafkaTemplate instances → 120K
> 3. **Add partitions:** 12 → 48 partitions → 120K
> 4. **Add brokers:** 3 → 6 brokers → 150K
>
> Enterprise at scale (Uber, LinkedIn) uses all four combined."

**Q: "What's the per-partition throughput?"**

> "30K events/sec ÷ 12 partitions = 2,500 events/sec per partition.
> Each partition is a single log on disk, so it's limited by disk I/O
> and network bandwidth of the broker it's on."

---

### Problem-Solving Questions

**Q: "What problems did you face?"**

> "Three main issues:
> 1. **Port conflict:** Zookeeper AdminServer uses 8080, same as Spring Boot. Fixed by changing to 8081.
> 2. **MeterRegistry bean missing:** Forgot to add spring-boot-starter-actuator dependency. Fixed by adding it.
> 3. **Dual listeners:** Docker containers couldn't communicate externally. Fixed by adding PLAINTEXT (internal) + HOST (localhost) listeners."

**Q: "How do you handle failed messages?"**

> "Three layers:
> 1. **Kafka retries:** Automatic with MAX_INT retries
> 2. **Producer callbacks:** Log failures with Micrometer metrics
> 3. **Dead Letter Queue:** For messages that fail after all retries
>
> In production, I'd add a database table for failed messages with retry logic."

**Q: "How do you monitor the pipeline?"**

> "Three monitoring layers:
> 1. **Micrometer metrics:** request count, failure rate, latency percentiles
> 2. **Kafka metrics:** consumer lag, partition distribution, broker health
> 3. **Grafana dashboards:** Real-time visualization, alerts on anomalies
>
> Key alerts: failure rate > 0.1%, consumer lag > 10K, p99 > 500ms."

---

## 8. Technology Stack Summary

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| Runtime | Java | 17 | Application runtime |
| Framework | Spring Boot | 3.5.3 | Web framework |
| Build | Gradle | 9.3.0 | Build tool |
| Messaging | Kafka | 3.x (Confluent 7.6.0) | Event streaming |
| Monitoring | Micrometer + Prometheus | Latest | Metrics |
| Load Testing | k6 | 2.1.0 | Performance testing |
| Containerization | Docker Compose | Latest | Infrastructure |
| OS | macOS | Sonoma | Development |

---

## 9. Key Metrics to Remember

```
┌─────────────────────────────────────────────────────────────┐
│  INTERVIEW CHEAT SHEET                                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Throughput:     30,000 events/sec (ceiling)               │
│  Per Partition:  2,500 events/sec                           │
│  Latency p95:   ~100ms (at ceiling)                        │
│  Latency p99:   ~150ms (at ceiling)                        │
│  Failures:      0% at 30K, 4.8% at 40K                    │
│  Partitions:    12 (consumer constraint)                    │
│  Replication:   3 (one per broker)                          │
│  Batch Size:    32KB                                        │
│  Linger:        10ms                                        │
│  Compression:   lz4 (~60% reduction)                        │
│  Buffer:        32MB                                        │
│  Acks:          all (zero data loss)                        │
│  Idempotence:   true (exactly-once)                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 10. Interview Flow Template

### Opening (30 seconds)

> "I built an event-driven pipeline for a chat messaging service. It ingests messages via REST API and publishes them to Kafka with exactly-once semantics, achieving 30K events/sec with zero data loss."

### Architecture (1 minute)

> "The architecture is: HTTP Client → REST Controller → Service Layer → KafkaTemplate (async) → Producer Buffer → Sender Thread → Kafka Cluster (3 brokers, 12 partitions).
>
> The key design decision was making KafkaTemplate.send() async — the HTTP layer returns immediately while the sender thread handles batching in the background."

### Configuration (1 minute)

> "Critical configs: acks=all for zero data loss, batch.size=32KB with linger.ms=10 for efficient batching, lz4 compression reducing network I/O by 60%, and idempotence enabled for exactly-once semantics."

### Throughput (1 minute)

> "Using k6 ramp testing, I found the ceiling at 30K events/sec. The bottleneck is the single sender thread — it can only batch and send ~30K messages/sec. Beyond that, the producer buffer fills up and we get BufferExhaustedException."

### Scaling (30 seconds)

> "To scale to 50K+, I'd increase batch.size to 128KB and linger.ms to 30ms. For 100K+, I'd add producer instances or scale partitions. Enterprise systems like Uber use all four approaches combined."

### Challenges (30 seconds)

> "Three main challenges: port conflicts (Zookeeper uses 8080), Docker networking (needed dual listeners), and finding the ceiling (ramp testing with k6). Each required understanding the underlying system, not just the framework."

---

## 11. Common Follow-Up Questions

| Question | Quick Answer |
|----------|--------------|
| "Why not RabbitMQ?" | "Kafka is better for high-throughput, durable event streaming. RabbitMQ is better for task queues with complex routing." |
| "Why not Pulsar?" | "Kafka is more mature, better ecosystem, easier to operate. Pulsar is newer but less battle-tested." |
| "How do you handle ordering?" | "Kafka guarantees ordering within a partition. We use conversationId as partition key, so all messages for a conversation go to the same partition." |
| "What about consumer lag?" | "Consumers poll at their own pace. If lag grows, we scale consumers or partitions." |
| "How do you test?" | "Four-phase testing: baseline (correctness), ramp (find ceiling), sustained (stability), chaos (resilience)." |
| "What about schema evolution?" | "Use Confluent Schema Registry with Avro. Allows schema changes without breaking consumers." |

---

## 12. Diagrams for Whiteboard

### Quick Architecture (30 seconds to draw)

```
┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐
│ Client │───►│  REST  │───►│Service │───►│Producer│───►│ Kafka  │
│  HTTP  │    │  API   │    │ Layer  │    │ Async  │    │Cluster │
└────────┘    └────────┘    └────────┘    └────────┘    └────────┘
                                  │
                                  ▼
                           ┌────────────┐
                           │  Callback  │
                           │  (Metrics) │
                           └────────────┘
```

### Data Flow (1 minute to draw)

```
1. HTTP POST /api/v1/messages
         │
         ▼
2. Controller validates
         │
         ▼
3. Service creates DTO
         │
         ▼
4. KafkaTemplate.send() ──► Returns immediately (2ms)
         │
         ▼
5. Sender Thread batches (10ms linger)
         │
         ▼
6. Batch sent to Kafka
         │
         ▼
7. Kafka writes + replicates (RF=3)
         │
         ▼
8. Callback fires (success/failure)
```

---

## 13. Closing Statement

> "This project taught me that building a high-throughput event pipeline isn't just about the framework — it's about understanding the underlying systems: how Kafka batches messages, how network I/O affects throughput, and how to systematically find and push bottlenecks. The 30K ceiling wasn't a limitation — it was a measurable, tunable parameter that I can scale based on business needs."

---

*Last updated: July 2025*
*Pipeline: kafka-pipeline (Spring Boot 3.5.3 + Kafka 3.x)*
