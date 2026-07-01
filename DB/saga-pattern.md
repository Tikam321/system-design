# Saga Pattern (Interview Notes)

## What is the Saga Pattern?

The **Saga Pattern** is a microservice design pattern used to maintain **data consistency** across multiple services **without using distributed transactions (2PC)**.

Instead of one global transaction, each microservice executes its **own local transaction**. If any step fails, **compensating transactions** are executed to undo the previously completed business operations.

> **Key Point:** Saga provides **Eventual Consistency**, not ACID consistency.

---

# Why do we need Saga?

In a microservice architecture, each service owns its own database.

Example:

```text
Order Service      -> OrderDB
Payment Service    -> PaymentDB
Inventory Service  -> InventoryDB
Shipping Service   -> ShippingDB
```

Since there is no shared database, we cannot use a single database transaction across all services.

Without Saga:

```text
Create Order      ✅
Charge Payment    ✅
Reserve Inventory ❌
```

Result:

- Customer is charged.
- Order is created.
- Inventory is not reserved.

The system becomes inconsistent.

Saga solves this by executing **compensating transactions**.

---

# Example (E-Commerce)

## Successful Flow

```text
Customer
    │
    ▼
Create Order
    │
    ▼
Charge Payment
    │
    ▼
Reserve Inventory
    │
    ▼
Create Shipment
    │
    ▼
Order Completed
```

---

## Failure Flow

```text
Create Order        ✅
Charge Payment      ✅
Reserve Inventory   ❌
```

Compensation:

```text
Refund Payment
      │
      ▼
Cancel Order
```

Instead of rolling back the database transaction, Saga performs business operations that reverse the completed work.

---

# Saga Approaches

There are two ways to implement Saga:

1. Choreography
2. Orchestration

---

# 
