# 1. What is the purpose of Docker in microservices deployment?
- Docker packages your application with all dependencies so it runs the same everywhere.
- Instead of saying it works on my machine, Docker gives a predictable runtime environment.
- In microservices, each service can be containerized and deployed independently.
- It also helps with scaling because orchestration tools like Kubernetes can spin up multiple containers easily.
- Docker images are versioned, so rollbacks and promotions between dev, QA, and prod become smoother.
- In real projects, Docker is basically the standard way to ship microservices reliably.

# 2. How do you handle distributed transactions in microservices?
- In microservices, we generally avoid distributed transactions because they're slow and fragile.
- Instead, we use patterns like Saga where each service does a local transaction and publishes an event.
- If something fails later, we run compensating actions to undo previous steps.
- This gives eventual consistency rather than strict ACID across services.
- We also use outbox pattern so DB changes and events are reliably published together.
- In real projects, distributed consistency is handled through design, not through one big transaction across services.

# 3. How do you monitor application performance in production?
- In production, we monitor metrics like CPU, memory, request latency, error rate, and throughput.
- We usually expose metrics using Micrometer in Spring Boot and send them to Prometheus/Grafana.
- We also track application logs centrally using ELK or similar, so debugging is fast.
- For microservices, distributed tracing is key - tools like OpenTelemetry with Jaeger/Zipkin help track a request end-to-end.
- We set up alerts on SLA metrics like p95 latency and error spikes.
- And we also monitor JVM health - GC pauses, heap usage, thread counts — because that's where many issues start.

# 4. What is caching and how have you implemented it in projects?
- Caching means storing frequently used data in faster storage so you don't hit database or external services again and again.
- In Spring Boot, a common approach is using Spring Cache with annotations like @Cacheable, @CachePut, and @CacheEvict.
- For simple cases, in-memory cache works, but in real production we mostly use Redis for distributed caching.
- We cache things like master data, configuration, user profile lookups, and expensive computed responses.
- But you must handle cache invalidation properly - otherwise you serve stale data.
- So we define TTLs, eviction rules, and clear cach
