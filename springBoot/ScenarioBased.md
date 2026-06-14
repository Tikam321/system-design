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
