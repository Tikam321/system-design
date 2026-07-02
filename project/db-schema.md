# Admin Portal – Hierarchical Policy Management (Interview Notes)

## Project Overview

I worked on the **Admin Portal**, a Spring Boot–based microservice responsible for managing enterprise policy configurations and administrative access. The system supports **hierarchical policy inheritance**, allowing policies to be configured at multiple levels.

### Policy Priority

```text
Default (Lowest Priority)
        │
        ▼
Company
        │
        ▼
Organization
        │
        ▼
User (Highest Priority)
```

The effective policy value is determined by traversing the hierarchy from **User → Organization → Company → Default**. The first available override becomes the effective value.

---

# Database Schema

The system uses the following core tables:

## 1. Policy Rule (Master Table)

Stores metadata for every policy.

**Columns (Example):**

* `policy_id`
* `policy_name`
* `policy_type`
* `validation_rule`
* `description`

This table contains **only policy metadata** and acts as the master reference for all policy configurations.

---

## 2. Company License Policy

Stores the policies available to a company based on its purchased license.

---

## 3. Company Policy Rule

Stores the default policy values configured for a company.

Example:

```text
Company A

Camera = Enabled
Screen Capture = Disabled
```

---

## 4. Organization Policy Rule

Stores policy overrides for a specific organization or sub-organization.

Example:

```text
Company Default

Camera = Enabled

Organization Override

Camera = Disabled
```

---

## 5. User Policy Rule

Stores user-specific policy overrides.

This is the **highest priority** level.

Example:

```text
Company

Camera = Enabled

Organization

Camera = Disabled

User

Camera = Enabled
```

Effective value = **Enabled**

---

# Table Relationships

```text
                  Policy Rule
                 (Master Table)
                      │
      ┌───────────────┼────────────────┐
      ▼               ▼                ▼
Company Policy   Organization Policy   User Policy
       │                 │                 │
       └────── References Policy Rule ────┘
```

All policy tables reference **Policy Rule** using a foreign key (`policy_id`).

---

# Why We Normalized the Schema

Instead of storing policy metadata in every table, we stored it only once in the **Policy Rule** table.

### Before Normalization

```text
Company Policy
---------------
Policy Name
Policy Type
Description
Value

Organization Policy
-------------------
Policy Name
Policy Type
Description
Value

User Policy
-----------
Policy Name
Policy Type
Description
Value
```

The same metadata would be duplicated across multiple tables.

### After Normalization

```text
Policy Rule
------------
Policy ID
Policy Name
Policy Type
Description

Company Policy Rule
-------------------
Company ID
Policy ID (FK)
Value

Organization Policy Rule
-------------------------
Organization ID
Policy ID (FK)
Value

User Policy Rule
----------------
User ID
Policy ID (FK)
Value
```

### Benefits

* Eliminated data redundancy.
* Stored policy metadata only once.
* Improved data consistency.
* Avoided update anomalies.
* Made adding new policies easier without changing multiple tables.
* Improved maintainability.

---

# Indexing

To optimize policy lookups, indexes were created on frequently queried columns such as:

* `company_id`
* `organization_id`
* `user_id`
* `policy_id`

This reduced query latency during policy evaluation.

---

# API Design

Initially, the React frontend had to invoke multiple REST APIs to fetch policy values from different hierarchy levels.

Example:

```text
GET /company-policy
GET /organization-policy
GET /user-policy
GET /default-policy
```

This resulted in:

* Multiple network calls.
* Increased frontend complexity.
* Higher response time.

To solve this, I designed a consolidated API:

```http
GET /api/stored/policy
```

The backend:

1. Retrieved policy values from all hierarchy levels.
2. Applied the precedence rule:

```text
User
   ↓
Organization
   ↓
Company
   ↓
Default
```

3. Returned the final effective policy values in a single response.

### Benefits

* Reduced multiple API calls to a single request.
* Improved page load time.
* Reduced frontend complexity.
* Centralized policy resolution logic.
* Ensured consistent policy evaluation across all clients.

---

# Design Trade-offs

### Advantages

* Normalized and maintainable schema.
* Flexible policy inheritance.
* Easy to introduce new policies.
* Improved API performance.
* Reduced network overhead.

### Trade-offs

* Policy retrieval requires checking multiple tables.
* Backend aggregation logic is more complex than a denormalized design.
* Slightly more expensive reads, but significantly better maintainability and data consistency.

---

# Interview Summary

> I worked on an Admin Portal microservice that managed hierarchical enterprise policies with the precedence **Default → Company → Organization → User**. I helped design a normalized database schema where policy metadata was stored in a master `PolicyRule` table and policy values were stored separately for each hierarchy level. This reduced data redundancy, improved consistency, and simplified maintenance. I also designed a consolidated REST API (`GET /api/stored/policy`) that aggregated policy values across all levels and returned the effective policy in a single response, reducing frontend API calls and improving application performance.
