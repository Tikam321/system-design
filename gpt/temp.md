# System Design Interview Preparation

## 1. Build the foundation

- Learn the common building blocks: load balancers, CDNs, caches, databases, message queues, object storage, and search indexes.
- Know the trade-offs among SQL vs. NoSQL, vertical vs. horizontal scaling, and synchronous vs. asynchronous processing.
- Review core concepts: latency, throughput, availability, consistency, durability, partitioning, replication, sharding, and back-pressure.

## 2. Use a repeatable interview framework

1. Clarify functional requirements, non-functional requirements, users, and scale.
2. Estimate traffic, storage, bandwidth, and peak load with rough calculations.
3. Sketch the high-level architecture and identify the request/data flow.
4. Design storage and APIs; state key data models and access patterns.
5. Deep-dive into the bottleneck most relevant to the prompt.
6. Explain reliability, monitoring, security, and trade-offs.
7. Summarize the final design and invite follow-up questions.

## 3. Practice core problems

Work through each problem aloud and draw the design:

- URL shortener
- Rate limiter
- News feed or timeline
- Chat or messaging system
- File storage and sharing
- Video streaming service
- Search autocomplete
- Notification service
- Web crawler
- Ride-sharing or food delivery dispatch

## 4. Strengthen communication

- Start broad, then deepen only where the interviewer directs you.
- Narrate assumptions and quantify them.
- State alternatives and why you selected one.
- Call out failure modes and mitigations before being asked.
- Keep diagrams simple: clients, edge, services, storage, and asynchronous workers.

## 5. Prepare a study schedule

### Week 1: Fundamentals

Study distributed-systems concepts and create a one-page cheat sheet for each component.

### Week 2: Storage and scale

Practice database selection, caching, replication, partitioning, and capacity estimation.

### Week 3: End-to-end designs

Complete one medium-difficulty design per day using the framework above.

### Week 4: Mock interviews

Run timed 45-minute mocks, record gaps, and revisit weak topics.

## 6. Final interview checklist

- Clarified scope and scale
- Estimated capacity
- Defined APIs and data model
- Chosen components with trade-offs
- Addressed bottlenecks and failure handling
- Covered observability and security
- Summarized decisions clearly
