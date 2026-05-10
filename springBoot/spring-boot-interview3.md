# Challenges When Migrating Monolith to Microservices

# 1. Distributed System Complexity

In monolith:

```text
Everything runs in one application
```

In microservices:

```text
Multiple services communicate over network
```

Challenges:
- Network latency
- Service failures
- Retry handling

---

# 2. Data Management

Monolith usually has:

```text
Single database
```

Microservices prefer:

```text
Database per service
```

Challenges:
- Data consistency
- Distributed transactions
- Data duplication

---

# 3. Service Communication

Need communication using:

- REST
- Kafka
- gRPC

Challenges:
- Timeout handling
- Service discovery
- API versioning

---

# 4. Deployment Complexity

Now multiple services need:
- CI/CD
- Containerization
- Monitoring

Deployment becomes harder.

---

# 5. Distributed Transactions

ACID transactions across services become difficult.

Need:
- Saga pattern
- Event-driven architecture

---

# 6. Monitoring and Debugging

Harder to trace requests across services.

Need:
- Centralized logging
- Distributed tracing
- Monitoring tools

---

# 7. Security Challenges

Need:
- JWT/OAuth2
- API Gateway
- Secure inter-service communication

---

# 8. Performance Issues

More network calls can increase:
- Latency
- Resource usage

---

# 9. Testing Complexity

Integration and end-to-end testing become more difficult.

---

# 10. Team and Service Boundaries

Improper service splitting can cause:
- Tight coupling
- Chatty communication
- Dependency issues

---

# Interview Ready Answer

> Migrating from monolith to microservices introduces challenges like distributed system complexity, service communication, data consistency, distributed transactions, deployment complexity, monitoring, and security. Managing inter-service communication, database separation, and debugging across multiple services becomes more difficult and requires tools like API Gateway, service discovery, centralized logging, and event-driven patterns.
