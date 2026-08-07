# SQL Interview Questions & Answers

A complete guide to SQL interview questions — covering fundamentals, joins, indexing, normalization, transactions, window functions, query optimization, and common coding problems.

---

## Table of Contents
1. [SQL Basics](#sql-basics)
2. [Joins](#joins)
3. [Aggregation & Grouping](#aggregation--grouping)
4. [Subqueries & CTEs](#subqueries--ctes)
5. [Window Functions](#window-functions)
6. [Indexes](#indexes)
7. [Normalization & Database Design](#normalization--database-design)
8. [Transactions & Concurrency](#transactions--concurrency)
9. [Constraints & Keys](#constraints--keys)
10. [Query Optimization & Performance](#query-optimization--performance)
11. [SQL vs NoSQL](#sql-vs-nosql)
12. [Common Coding Questions](#common-coding-questions)
13. [Advanced / Scenario-Based Questions](#advanced--scenario-based-questions)

---

## SQL Basics

### 1. What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?

| | `DELETE` | `TRUNCATE` | `DROP` |
|---|---|---|---|
| Removes | Specific rows (with `WHERE`) or all rows | All rows | The entire table structure + data |
| Rollback | Can be rolled back (logged, DML) | Usually cannot be rolled back (DDL, minimal logging) | Cannot be rolled back |
| Speed | Slower (row-by-row logging) | Faster (deallocates pages, doesn't log each row) | Fastest (removes object entirely) |
| Triggers | Fires `DELETE` triggers | Does not fire triggers (in most DBs) | N/A |
| Resets identity/auto-increment | No | Yes (typically) | N/A (table gone) |

### 2. What is the difference between `WHERE` and `HAVING`?
- **`WHERE`** filters rows **before** grouping/aggregation — cannot reference aggregate functions.
- **`HAVING`** filters groups **after** `GROUP BY`/aggregation — used specifically to filter on aggregate results.

```sql
SELECT department, COUNT(*) AS emp_count
FROM employees
WHERE status = 'active'         -- filters rows first
GROUP BY department
HAVING COUNT(*) > 5;            -- filters groups after aggregation
```

### 3. What is the order of execution of a SQL query?
Logical execution order (not the order you write it in):
```
FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT/OFFSET
```
This is why you can't use a column alias (defined in `SELECT`) inside a `WHERE` clause — `WHERE` executes before `SELECT` is evaluated.

### 4. What is the difference between `UNION` and `UNION ALL`?
- **`UNION`** combines result sets and **removes duplicates** (requires an implicit sort/dedup, more expensive).
- **`UNION ALL`** combines result sets **keeping duplicates** — faster since no dedup step is needed.
Both require the combined queries to have the same number of columns with compatible data types.

### 5. What is the difference between `CHAR` and `VARCHAR`?
- **`CHAR(n)`** — fixed-length; always stores `n` characters, padding with spaces if shorter. Slightly faster for fixed-size data.
- **`VARCHAR(n)`** — variable-length; stores only the actual characters used (plus a small length header), up to a max of `n`. More space-efficient for variable-length data.

### 6. What is the difference between `IN` and `EXISTS`?
- **`IN`** compares a value against a list/subquery result — evaluates the full subquery result set first (can be a list of literals or a subquery).
- **`EXISTS`** checks only whether a subquery returns **any** rows at all (stops as soon as one match is found) — often faster for correlated subqueries with large datasets, since it doesn't need to materialize the full result set.

```sql
-- IN
SELECT * FROM orders WHERE customer_id IN (SELECT id FROM customers WHERE country = 'US');

-- EXISTS (often more efficient for correlated checks)
SELECT * FROM orders o
WHERE EXISTS (SELECT 1 FROM customers c WHERE c.id = o.customer_id AND c.country = 'US');
```

### 7. What is the difference between `NULL` and an empty string `''`?
`NULL` represents the **absence of a value** (unknown/undefined), while `''` is an actual empty string value. `NULL` is never equal to anything, including itself (`NULL = NULL` evaluates to `NULL`, not `TRUE`) — you must use `IS NULL`/`IS NOT NULL` to check for it.

### 8. What is the difference between `RANK()`, `DENSE_RANK()`, and `ROW_NUMBER()`?
Covered in detail in the [Window Functions](#window-functions) section below.

---

## Joins

### 9. Explain the different types of SQL joins.

- **INNER JOIN** — returns only rows with matching values in both tables.
- **LEFT (OUTER) JOIN** — returns all rows from the left table, and matched rows from the right (unmatched right columns are `NULL`).
- **RIGHT (OUTER) JOIN** — returns all rows from the right table, and matched rows from the left.
- **FULL (OUTER) JOIN** — returns all rows from both tables, matched where possible, `NULL` where not.
- **CROSS JOIN** — returns the Cartesian product of both tables (every row combined with every row).
- **SELF JOIN** — a table joined with itself, typically to compare rows within the same table (e.g., employee-manager relationships).

```sql
-- Self join example: find each employee's manager name
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### 10. What is the difference between `INNER JOIN` and `LEFT JOIN`?
`INNER JOIN` excludes rows that don't have a match in both tables. `LEFT JOIN` keeps every row from the left table regardless of a match, filling unmatched columns from the right table with `NULL`.

### 11. How do you find rows in one table that don't exist in another (anti-join)?
```sql
-- Using LEFT JOIN + IS NULL
SELECT c.*
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
WHERE o.id IS NULL;

-- Using NOT EXISTS (often more efficient)
SELECT c.*
FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
```

### 12. What is the difference between `NOT IN` and `NOT EXISTS`, and why does `NOT IN` behave unexpectedly with `NULL`s?
If the subquery in `NOT IN` returns even a single `NULL` value, the **entire query returns zero rows** — because `x NOT IN (1, 2, NULL)` evaluates each comparison, and `x <> NULL` is `NULL` (unknown), not `TRUE`, which poisons the whole `OR` chain. `NOT EXISTS` doesn't have this problem since it just checks row existence. **Best practice: prefer `NOT EXISTS` over `NOT IN` when the subquery column can contain `NULL`s.**

---

## Aggregation & Grouping

### 13. What are the common aggregate functions in SQL?
`COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()` — all ignore `NULL` values except `COUNT(*)`, which counts all rows regardless of `NULL`s.

### 14. What is the difference between `COUNT(*)`, `COUNT(1)`, and `COUNT(column_name)`?
- **`COUNT(*)`** counts all rows, including those with `NULL` values in any column.
- **`COUNT(1)`** behaves identically to `COUNT(*)` in virtually all modern optimizers (counts all rows) — historically thought to be faster, but this is a myth in modern engines.
- **`COUNT(column_name)`** counts only rows where that specific column is **not `NULL`**.

### 15. Can you use an aggregate function without `GROUP BY`?
Yes — if there's no `GROUP BY`, the aggregate function treats the **entire result set as a single group**, returning one row.

```sql
SELECT COUNT(*), AVG(salary) FROM employees; -- single summary row
```

### 16. What happens if you `SELECT` a non-aggregated column that isn't in the `GROUP BY` clause?
In strict SQL mode (and most modern databases like PostgreSQL, MySQL 5.7+ with `ONLY_FULL_GROUP_BY`), this throws an error, since the value would be ambiguous — which row's value should represent the whole group? Every non-aggregated selected column must appear in the `GROUP BY` clause.

### 17. What is `GROUP BY` with multiple columns?
Groups rows by the unique combination of all specified columns, not each independently.

```sql
SELECT department, job_title, COUNT(*)
FROM employees
GROUP BY department, job_title;
```

### 18. What is `ROLLUP` / `CUBE` used for?
- **`ROLLUP`** generates subtotal rows plus a grand total, following a hierarchy of the grouped columns (e.g., subtotal per department, then a grand total row).
- **`CUBE`** generates subtotals for **every possible combination** of the grouped columns, not just a hierarchy.

```sql
SELECT department, job_title, SUM(salary)
FROM employees
GROUP BY ROLLUP(department, job_title);
```

---

## Subqueries & CTEs

### 19. What is a subquery? What are the types?
A query nested inside another query. Types:
- **Scalar subquery** — returns a single value, usable anywhere a single value is expected.
- **Row subquery** — returns a single row with multiple columns.
- **Table subquery** — returns multiple rows/columns, used in `FROM` or with `IN`/`EXISTS`.
- **Correlated subquery** — references a column from the outer query, re-evaluated once per outer row (can be slow on large datasets).

```sql
-- Correlated subquery: employees earning above their department's average
SELECT e.name, e.salary, e.department
FROM employees e
WHERE e.salary > (
    SELECT AVG(e2.salary) FROM employees e2 WHERE e2.department = e.department
);
```

### 20. What is a CTE (Common Table Expression)? Why use it over a subquery?
A `WITH` clause that defines a named, temporary result set, referenceable within the main query — improves readability, allows reuse of the same result set multiple times in a query, and supports **recursion** (which plain subqueries cannot do).

```sql
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 100000
)
SELECT department, COUNT(*) FROM high_earners GROUP BY department;
```

### 21. What is a recursive CTE? Give an example.
A CTE that references itself, used for hierarchical/tree-structured data (org charts, category trees, graph traversal).

```sql
WITH RECURSIVE org_chart AS (
    -- anchor member: top-level managers
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- recursive member: join back to find each next level
    SELECT e.id, e.name, e.manager_id, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level;
```

### 22. What is the difference between a CTE and a temporary table / view?
- **CTE** — exists only for the duration of a single query, not stored, not indexable, recomputed each time it's referenced (unless the optimizer materializes it).
- **Temp table** — physically stored (in tempdb/session), can be indexed, persists for the session/transaction, better for reuse across multiple queries or when the intermediate result is large.
- **View** — a stored, named query definition (not the data itself), reusable across sessions, always reflects live underlying data.

---

## Window Functions

### 23. What is a window function? How is it different from `GROUP BY`?
A window function performs a calculation across a set of rows **related to the current row**, without collapsing the result into a single row per group — unlike `GROUP BY`, which reduces multiple rows into one summary row per group, window functions **keep all original rows** while adding a computed column.

```sql
SELECT name, department, salary,
       AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary
FROM employees;
-- every employee row is kept, with their department's average salary attached
```

### 24. What is the difference between `RANK()`, `DENSE_RANK()`, and `ROW_NUMBER()`?

| Function | Behavior on ties | Example (scores: 90, 90, 80) |
|---|---|---|
| `ROW_NUMBER()` | Assigns a unique, sequential number regardless of ties | 1, 2, 3 |
| `RANK()` | Same rank for ties, but **skips** the next rank number(s) | 1, 1, 3 |
| `DENSE_RANK()` | Same rank for ties, **no gaps** in the following rank | 1, 1, 2 |

```sql
SELECT name, score,
       ROW_NUMBER() OVER (ORDER BY score DESC) AS rn,
       RANK()       OVER (ORDER BY score DESC) AS rnk,
       DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rnk
FROM students;
```

### 25. What does `PARTITION BY` do in a window function?
Divides the result set into partitions (groups) — the window function is applied independently **within each partition**, similar to how `GROUP BY` segments data, but without collapsing rows.

### 26. What are `LAG()` and `LEAD()` used for?
Access data from a **previous** (`LAG`) or **next** (`LEAD`) row within the same result set, without a self-join — commonly used for period-over-period comparisons (e.g., month-over-month sales change).

```sql
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
       revenue - LAG(revenue) OVER (ORDER BY month) AS revenue_change
FROM monthly_sales;
```

### 27. How do you find the Nth highest salary using window functions?
```sql
SELECT DISTINCT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk = 3;  -- 3rd highest distinct salary
```
Using `DENSE_RANK()` (not `ROW_NUMBER()`) ensures duplicate salary values are treated as the same rank, avoiding skipped positions from ties.

### 28. What are frame clauses (`ROWS BETWEEN` / `RANGE BETWEEN`) in window functions?
They define which rows within the partition are included in the calculation relative to the current row — useful for running totals, moving averages.

```sql
SELECT order_date, amount,
       SUM(amount) OVER (
           ORDER BY order_date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS rolling_3day_total
FROM orders;
```

---

## Indexes

### 29. What is an index? Why does it speed up queries?
A separate data structure (typically a **B-Tree**, sometimes a hash or bitmap index) that stores a sorted reference to column values and pointers to the corresponding table rows — allowing the database to locate rows via a fast lookup/binary search instead of scanning every row (a full table scan).

### 30. What is the difference between a clustered and a non-clustered index?
- **Clustered index** — determines the **physical storage order** of the table's data rows. A table can have only **one** clustered index (often the primary key by default).
- **Non-clustered index** — a separate structure holding sorted key values and pointers back to the actual data rows (via the clustered index key or row ID). A table can have **many** non-clustered indexes.

### 31. What are the downsides of having too many indexes?
- Slower `INSERT`/`UPDATE`/`DELETE` operations, since every index must also be updated on every write
- Additional storage overhead
- Query optimizer may sometimes choose a suboptimal index, adding complexity to query planning

### 32. What is a composite (multi-column) index? What is the "leftmost prefix" rule?
An index built on multiple columns together. The **leftmost prefix rule** states that a composite index `(A, B, C)` can be used efficiently for queries filtering on `A`, `(A, B)`, or `(A, B, C)` — but **not** for queries filtering on `B` or `C` alone, since the index is sorted primarily by `A` first.

```sql
CREATE INDEX idx_name_dept ON employees(department, name);
-- Efficiently used by: WHERE department = 'Eng'
--                       WHERE department = 'Eng' AND name = 'Bob'
-- NOT efficiently used by: WHERE name = 'Bob'  (department not filtered)
```

### 33. When would an index NOT be used, even if one exists?
- Using a function/expression on the indexed column in the `WHERE` clause (e.g., `WHERE YEAR(order_date) = 2024` instead of a range on `order_date`) — unless a functional/expression index exists
- Leading wildcard `LIKE` searches (`LIKE '%something'`) — can't use a standard B-Tree index efficiently since it can't do a prefix scan
- Implicit type conversion between the column and the compared value
- Low selectivity — if the indexed column has very few distinct values (e.g., a boolean flag), the optimizer may prefer a full table scan over the index
- `OR` conditions spanning different columns without appropriate composite/multiple indexes

### 34. What is a covering index?
An index that contains **all the columns** needed to satisfy a query (both filter and selected columns), so the database can answer the query entirely from the index without needing to access the actual table rows — significantly faster.

### 35. What is the difference between a B-Tree index and a Hash index?
- **B-Tree** — sorted structure, supports range queries (`<`, `>`, `BETWEEN`), equality, and prefix `LIKE` searches. The default and most common index type.
- **Hash index** — extremely fast for exact equality lookups (`=`) but cannot support range queries or sorting, since hashed values have no meaningful order.

---

## Normalization & Database Design

### 36. What is database normalization? Why do we normalize?
The process of organizing data to reduce redundancy and avoid update/insert/delete anomalies, by splitting data into related tables and defining relationships via foreign keys.

### 37. Explain the normal forms (1NF, 2NF, 3NF, BCNF).
- **1NF (First Normal Form)**: each column holds atomic (indivisible) values; no repeating groups or arrays in a single column.
- **2NF**: must be in 1NF, and every non-key column must depend on the **entire** primary key (relevant when using composite keys) — no partial dependency.
- **3NF**: must be in 2NF, and no non-key column depends on another non-key column (no transitive dependency) — every non-key column depends only on the primary key.
- **BCNF (Boyce-Codd Normal Form)**: a stricter version of 3NF — every determinant (a column/set of columns that determines another column) must be a candidate key.

### 38. What is denormalization? When would you use it?
Deliberately introducing redundancy (e.g., duplicating a column across tables, or pre-computing aggregates) to improve read performance, at the cost of extra storage and more complex writes/updates. Common in reporting/analytics systems, read-heavy applications, or when join costs become a bottleneck at scale.

### 39. What is a foreign key? What does `ON DELETE CASCADE` do?
A foreign key enforces referential integrity — ensuring a column's value must exist in the referenced table's primary/unique key column. `ON DELETE CASCADE` automatically deletes dependent rows in the child table when the referenced row in the parent table is deleted (other options: `SET NULL`, `RESTRICT`, `NO ACTION`).

### 40. What is the difference between a primary key and a unique key?
| Primary Key | Unique Key |
|---|---|
| Only one per table | Multiple allowed per table |
| Cannot contain `NULL` | Can contain one `NULL` (in most DBs) |
| Automatically creates a clustered index (in most DBs) | Creates a non-clustered index by default |
| Uniquely identifies each row | Enforces uniqueness, but doesn't have to be "the" identifier |

---

## Transactions & Concurrency

### 41. What is a transaction? What are the ACID properties?
A transaction is a sequence of operations executed as a single logical unit of work, either fully completed or fully rolled back.
- **Atomicity** — all operations succeed, or none do (all-or-nothing).
- **Consistency** — the database moves from one valid state to another, respecting all constraints.
- **Isolation** — concurrent transactions don't interfere with each other's intermediate states.
- **Durability** — once committed, changes persist even after a crash/power failure.

### 42. What are the transaction isolation levels?
| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| **Read Uncommitted** | Possible | Possible | Possible |
| **Read Committed** | Prevented | Possible | Possible |
| **Repeatable Read** | Prevented | Prevented | Possible (varies by DB) |
| **Serializable** | Prevented | Prevented | Prevented |

Higher isolation = stronger consistency guarantees, but more locking overhead and reduced concurrency.

### 43. What is a dirty read, non-repeatable read, and phantom read?
- **Dirty read**: reading uncommitted data from another transaction that might later be rolled back.
- **Non-repeatable read**: reading the same row twice within a transaction and getting different values, because another transaction modified and committed it in between.
- **Phantom read**: re-running the same query within a transaction and getting a **different set of rows**, because another transaction inserted/deleted rows matching the query's criteria in between.

### 44. What is deadlock? How can it be avoided?
A situation where two or more transactions are each waiting on a lock held by the other, so none can proceed. Avoided by: acquiring locks in a **consistent order** across transactions, keeping transactions short, using appropriate isolation levels, and setting lock/deadlock timeouts so the database can detect and abort one of the transactions.

### 45. What is optimistic vs pessimistic locking?
- **Pessimistic locking**: acquires a lock on a row upfront (e.g., `SELECT ... FOR UPDATE`), blocking other transactions from modifying it until released — safer under high contention, but reduces concurrency.
- **Optimistic locking**: doesn't lock upfront; instead checks (usually via a version/timestamp column) at commit time whether the row was modified by someone else since it was read, and fails/retries if so — better for low-contention scenarios with higher throughput.

---

## Constraints & Keys

### 46. What are the common SQL constraints?
`PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`, `CHECK` (validates a condition, e.g., `CHECK (age >= 18)`), `DEFAULT` (sets a default value if none provided).

### 47. What is a composite key?
A primary key made up of two or more columns together, used when no single column uniquely identifies a row (common in many-to-many junction/link tables).

### 48. What is the difference between a natural key and a surrogate key?
- **Natural key**: a column with inherent business meaning that could uniquely identify a row (e.g., email, SSN, ISBN).
- **Surrogate key**: an artificial, system-generated identifier with no business meaning (e.g., an auto-incrementing `id` or UUID) — preferred in most designs since it's stable even if business data changes (e.g., an email can be updated; a surrogate ID never needs to).

---

## Query Optimization & Performance

### 49. How do you find and fix a slow query?
1. Use `EXPLAIN`/`EXPLAIN ANALYZE` to see the query execution plan — look for full table scans, missing index usage, expensive sort/join operations
2. Check if appropriate indexes exist for `WHERE`/`JOIN`/`ORDER BY` columns
3. Avoid `SELECT *` — retrieve only needed columns (reduces I/O, may allow a covering index)
4. Rewrite correlated subqueries as joins or `EXISTS` where possible
5. Check for implicit type conversions or functions on indexed columns preventing index usage
6. Consider denormalization or materialized views for expensive, frequently-run aggregate queries

### 50. What does `EXPLAIN` (or `EXPLAIN ANALYZE`) show you?
The query execution plan — how the database intends to execute the query: which indexes (if any) are used, join order and join algorithm (nested loop, hash join, merge join), estimated vs actual row counts, and where the most time/cost is spent. `EXPLAIN ANALYZE` actually runs the query and shows real execution stats, not just estimates.

### 51. What is the N+1 query problem?
Executing one query to fetch a list of records, then executing **one additional query per record** to fetch related data (e.g., fetching 100 orders, then querying each order's customer separately = 101 queries total). Fixed by using a `JOIN`, batch fetching, or ORM-specific solutions (`JOIN FETCH`, `@EntityGraph` in JPA/Hibernate).

### 52. What is a materialized view? How is it different from a regular view?
A **materialized view** physically stores the query result on disk (like a cached snapshot), which must be manually or periodically refreshed — much faster to read since the data doesn't need to be recomputed each time. A **regular view** is just a stored query definition, executed fresh (against live data) every time it's queried.

### 53. What is query plan caching, and why can it sometimes cause problems?
Databases cache execution plans for repeated queries to avoid re-planning overhead. This can cause **parameter sniffing** issues — a plan optimized for one parameter's data distribution gets reused for a very different parameter value, leading to a suboptimal plan for that case.

---

## SQL vs NoSQL

### 54. When would you choose SQL over NoSQL, and vice versa?
- **SQL (relational)**: structured data with clear relationships, strong consistency (ACID) requirements, complex queries/joins, well-defined schema — e.g., financial transactions, inventory systems.
- **NoSQL**: flexible/evolving schema, massive horizontal scale, high write throughput, denormalized/document-oriented access patterns, eventual consistency acceptable — e.g., session storage, catalogs with varying attributes, high-velocity event data.

### 55. What is the CAP theorem, and how does it relate to database choice?
States that a distributed system can only guarantee **two** of the following three at once: **Consistency** (every read gets the latest write), **Availability** (every request gets a response), **Partition tolerance** (the system keeps working despite network partitions). Since partition tolerance is generally non-negotiable in distributed systems, the real-world tradeoff is usually **CP vs AP** — most relational databases favor consistency, many NoSQL databases (e.g., Cassandra, DynamoDB) let you tune this per-use-case.

---

## Common Coding Questions

### 56. Find duplicate rows in a table.
```sql
SELECT email, COUNT(*)
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

### 57. Delete duplicate rows, keeping only one copy.
```sql
DELETE FROM users
WHERE id NOT IN (
    SELECT MIN(id)
    FROM users
    GROUP BY email
);
```

### 58. Find the second highest salary.
```sql
-- Using LIMIT/OFFSET
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- Using a subquery (portable across most DBs)
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

### 59. Find employees who earn more than their manager.
```sql
SELECT e.name AS employee, e.salary, m.name AS manager, m.salary AS manager_salary
FROM employees e
JOIN employees m ON e.manager_id = m.id
WHERE e.salary > m.salary;
```

### 60. Find the department with the highest average salary.
```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC
LIMIT 1;
```

### 61. Write a query to pivot rows into columns (e.g., total sales per product per quarter).
```sql
SELECT product,
       SUM(CASE WHEN quarter = 'Q1' THEN sales ELSE 0 END) AS Q1,
       SUM(CASE WHEN quarter = 'Q2' THEN sales ELSE 0 END) AS Q2,
       SUM(CASE WHEN quarter = 'Q3' THEN sales ELSE 0 END) AS Q3,
       SUM(CASE WHEN quarter = 'Q4' THEN sales ELSE 0 END) AS Q4
FROM sales
GROUP BY product;
```

### 62. Find consecutive days a user was active (gaps-and-islands problem).
```sql
WITH numbered AS (
    SELECT user_id, activity_date,
           ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY activity_date) AS rn
    FROM user_activity
),
grouped AS (
    SELECT user_id, activity_date,
           DATE_SUB(activity_date, INTERVAL rn DAY) AS grp
    FROM numbered
)
SELECT user_id, MIN(activity_date) AS streak_start, MAX(activity_date) AS streak_end,
       COUNT(*) AS streak_length
FROM grouped
GROUP BY user_id, grp
ORDER BY user_id, streak_start;
```

### 63. Find the running total of sales by date.
```sql
SELECT order_date, amount,
       SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM sales;
```

### 64. Swap values of two columns without using a temp column.
```sql
UPDATE employees
SET first_name = last_name,
    last_name = first_name;
-- Most SQL engines evaluate the SET clause using original row values, so this works safely
```

---

## Advanced / Scenario-Based Questions

### 65. How would you design a schema for a many-to-many relationship?
Use a **junction/associative table** containing foreign keys to both related tables, often with its own composite primary key (or surrogate key) and any relationship-specific attributes.

```sql
CREATE TABLE student_courses (
    student_id INT REFERENCES students(id),
    course_id INT REFERENCES courses(id),
    enrolled_date DATE,
    PRIMARY KEY (student_id, course_id)
);
```

### 66. How do you handle pagination efficiently for large datasets?
- **`OFFSET`/`LIMIT`** is simple but gets progressively slower for deep pages, since the database still has to scan/skip all preceding rows.
- **Keyset (cursor-based) pagination** is more efficient at scale: instead of `OFFSET`, filter using the last seen row's sort key.

```sql
-- Keyset pagination: fetch next page after id = 1000
SELECT * FROM orders
WHERE id > 1000
ORDER BY id
LIMIT 20;
```

### 67. How would you prevent SQL injection?
Always use **parameterized queries / prepared statements** instead of string-concatenating user input into SQL. Never trust or directly embed raw user input into a query string. ORMs (JPA/Hibernate, MyBatis with `#{}` binding) handle this automatically when used correctly — but raw string concatenation (or MyBatis's `${}` substitution) bypasses this protection and remains vulnerable.

### 68. What's the difference between horizontal and vertical partitioning/sharding?
- **Vertical partitioning**: splitting a table by columns — e.g., moving rarely-used or large columns (like a `bio` text field) into a separate table to keep the main table lean.
- **Horizontal partitioning (sharding)**: splitting a table by rows — e.g., distributing rows across multiple physical tables/servers based on a key (date range, hash of a customer ID), used to scale beyond a single machine's capacity.

### 69. How would you design a table to efficiently store and query time-series data?
- Partition by time range (e.g., monthly/daily partitions) so queries filtering by date only scan relevant partitions
- Index on the timestamp column (often as part of a composite index with a device/entity ID)
- Consider a purpose-built time-series database (TimescaleDB, InfluxDB) if volume/query patterns justify it, rather than forcing it into a general relational schema

### 70. What is a common table pitfall when designing for soft deletes (`is_deleted` flag)?
Every query in the application must remember to filter `WHERE is_deleted = false` — easy to forget, leading to "deleted" data leaking into results. Common mitigations: use a database **view** that already filters out deleted rows, or a **partial/filtered index** on `is_deleted = false` to both enforce and speed up the common case.

---

## Quick Cheat-Sheet Summary

| Concept | Key Point |
|---|---|
| `WHERE` vs `HAVING` | Filters rows before vs groups after aggregation |
| Query execution order | `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY` |
| `IN` vs `EXISTS` | `EXISTS` often faster for correlated subqueries; `NOT IN` breaks with `NULL`s |
| Clustered vs non-clustered index | One per table (physical order) vs many (separate lookup structure) |
| Leftmost prefix rule | Composite index `(A,B,C)` only helps queries filtering from `A` onward |
| `RANK` vs `DENSE_RANK` vs `ROW_NUMBER` | Gaps on ties / no gaps on ties / always unique |
| ACID | Atomicity, Consistency, Isolation, Durability |
| N+1 problem | 1 query + N per-row queries → fix with joins/batch fetch |
| Optimistic vs pessimistic locking | Check-at-commit vs lock-upfront |
| `OFFSET` vs keyset pagination | Keyset scales better for deep pages |

---

*Good luck with your interview! A very common follow-up pattern: interviewers give you a schema and ask you to write a query live — practice explaining your query's logic out loud as you write it, not just producing the final SQL.*
