# Event-Driven Chat Messaging Pipeline — Architecture Notes

Reference notes for a Kafka-based chat messaging pipeline targeting 50,000 messages/sec.

## Goal

Every user message becomes an event pushed to Kafka. Messages are partitioned by `chatId`, so all messages belonging to a single conversation land on the same partition — preserving order within that chat.

## Configuration under evaluation

| Setting | Value |
|---|---|
| Partition key | `chatId` |
| Partition count | 12 |
| Application worker threads | 250 |
| `batch.size` | 16 KB |
| `linger.ms` | 5 ms |
| `compression.type` | LZ4 |
| Send pattern | Non-blocking, `CompletableFuture` + callback |
| Target throughput | 50,000 msgs/sec |

## Why keying by `chatId` is correct

Kafka only guarantees ordering **within a partition**. Keying by `chatId` ensures every message in a given conversation is delivered to the same partition, so consumers always see that chat's messages in send order. This is the right primitive for a chat system — you cannot have message 2 processed before message 1 in the same thread.

**Trade-off to watch:** hot partitions. If one chat (a large group, a broadcast channel) generates disproportionate traffic, all of its messages still funnel through a single partition — you can't parallelize a single ordered key across partitions. With 12 partitions this is fine for typical chat distributions, but monitor per-partition throughput (Grafana) to catch skew early.

## Throughput math

Assuming ~200-byte average message size (typical text message + metadata):

**Per-partition load** (even distribution assumed):
```
50,000 msgs/sec ÷ 12 partitions ≈ 4,167 msgs/sec/partition
```

**Batch fill time vs. linger.ms:**
```
16 KB batch ÷ 200 bytes/msg ≈ 82 messages to fill a batch
82 messages ÷ 4,167 msgs/sec ≈ 19.7 ms to fill a batch
```

Since `linger.ms = 5` is shorter than the ~19.7 ms fill time, batches are **timer-bound, not size-bound** at this rate — every 5 ms you flush whatever has accumulated (~21 messages, ~4 KB), not the full 16 KB. This isn't a problem: the 16 KB batch size acts as a ceiling that protects against spikes, rather than being the steady-state behavior.

**Verifying the math closes:**
```
200 batches/sec/partition × 12 partitions × ~21 msgs/batch ≈ 50,400 msgs/sec
```

✅ **50k msgs/sec is achievable** with this config, assuming:
- ~200-byte average message size (bigger messages make batching even more efficient, since batches fill by size before linger fires)
- Healthy broker disks (SSD)
- No network saturation
- Reasonably even traffic distribution across the 12 partitions

## Thread pool — one important clarification

Be precise about what the 250 threads represent, since it changes the design:

- **If they're application-level worker/ingestion threads** (handling inbound requests, validation, enrichment, then calling `producer.send()`) — this is a reasonable pool size for a high-concurrency chat backend. ✅
- **If they're meant to be 250 separate `KafkaProducer` instances** — this is inefficient. A `KafkaProducer` is thread-safe and designed to be shared across many calling threads. It has **one internal I/O ("Sender") thread** that does the actual batching and network writes, regardless of how many application threads call `.send()`. Spinning up 250 producer instances multiplies connection overhead, memory (each producer has its own buffer pool), and broker-side connection load with no throughput benefit.

**Recommended pattern:** 1 shared producer instance (or a small pool of 2–4 for extreme load), fed by the 250-thread application worker pool. `send()` is non-blocking — it hands the record to the internal accumulator and returns a `Future` immediately, so worker threads aren't waiting on network I/O.

Wrapping the result in `CompletableFuture` for callback chaining (instead of a blocking `.get()`) is the right pattern — just keep callback logic lightweight, since it runs on the producer's I/O thread, and a slow callback stalls all subsequent sends on that producer.

## Ordering safety net: idempotence

Because message order matters per chat, add:

```properties
enable.idempotence=true
```

Without this, if a batch fails and retries, `max.in.flight.requests.per.connection > 1` can reorder messages within a partition during failure/retry conditions — silently breaking the `chatId` ordering guarantee exactly when things go wrong.

## Pipeline diagram (text form)

```
Chat users send messages
          │
          ▼
┌─────────────────────────────────────────────┐
│ Ingestion service                            │
│ 250-thread pool calls the shared producer    │
│                                               │
│  ┌───────────────────┐  ┌───────────────────┐│
│  │ Worker thread pool│  │ Kafka producer     ││
│  │ 250 threads,      │  │ client             ││
│  │ async send()      │  │ 16KB batch, 5ms    ││
│  │                   │  │ linger, LZ4        ││
│  └───────────────────┘  └───────────────────┘│
└─────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────┐
│ Kafka topic: chat-messages                   │
│ 12 partitions, keyed by chatId hash          │
│                                               │
│  [P0][P1][P2][P3][P4][P5][P6][P7][P8][P9]    │
│  [P10][P11]                                  │
└─────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────┐
│ Consumer group                               │
│ Real-time analytics service                  │
└─────────────────────────────────────────────┘
```

## Summary checklist

- [x] Partition key = `chatId` → correct for per-chat ordering
- [x] 12 partitions → sufficient for 50k/sec at ~200-byte messages
- [x] `batch.size=16KB`, `linger.ms=5` → timer-bound at this rate, ceiling protects spikes
- [x] LZ4 compression → good speed/ratio trade-off for high throughput
- [x] Async send + callback → correct, non-blocking
- [ ] **Clarify**: 250 threads = application workers, not 250 producer instances
- [ ] **Add**: `enable.idempotence=true` to protect ordering during retries
- [ ] **Monitor**: per-partition throughput for hot-chat skew
