# 1. What is the purpose of Docker in microservices deployment?
- Docker packages your application with all dependencies so it runs the same everywhere.
- Instead of saying it works on my machine, Docker gives a predictable runtime environment.
- In microservices, each service can be containerized and deployed independently.
- It also helps with scaling because orchestration tools like Kubernetes can spin up multiple containers easily.
- Docker images are versioned, so rollbacks and promotions between dev, QA, and prod become smoother.
- In real projects, Docker is basically the standard way to ship microservices reliably.
