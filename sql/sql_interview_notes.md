# SQL Interview Notes (Chapters 1--2)

## Chapter 1: Basic SQL

### Sample Table

    id name      salary department     age
  ---- ------- -------- ------------ -----
     1 John       50000 IT              25
     2 David      60000 HR              30
     3 Emma       70000 IT              35
     4 Lisa       45000 Finance         28
     5 Mike       90000 IT              40

## Topics Covered

-   SELECT
-   WHERE
-   ORDER BY
-   LIMIT
-   DISTINCT
-   BETWEEN
-   IN
-   LIKE (`%`, `_`)
-   Basic Subquery

### Common Syntax

``` sql
SELECT * FROM Employee;

SELECT name, salary
FROM Employee
WHERE salary > 50000
ORDER BY salary DESC
LIMIT 3;
```

### LIKE Examples

``` sql
-- Starts with J
WHERE name LIKE 'J%'

-- Ends with n
WHERE name LIKE '%n'

-- Contains i
WHERE name LIKE '%i%'

-- Exactly 5 letters
WHERE name LIKE '_____'
```

### Important Notes

-   `WHERE` filters rows.
-   `ORDER BY ASC` is default.
-   `LIMIT n` returns first n rows.
-   `DISTINCT` removes duplicates.
-   `BETWEEN` is inclusive.

------------------------------------------------------------------------

## Chapter 2: Aggregate Functions & GROUP BY

### Aggregate Functions

``` sql
COUNT(*)
COUNT(column)
COUNT(DISTINCT column)
SUM(column)
AVG(column)
MIN(column)
MAX(column)
```

### GROUP BY

``` sql
SELECT department,
       AVG(salary)
FROM Employee
GROUP BY department;
```

### HAVING

``` sql
SELECT department,
       AVG(salary)
FROM Employee
GROUP BY department
HAVING AVG(salary) > 60000;
```

### WHERE vs HAVING

  WHERE                            HAVING
  -------------------------------- -----------------------------
  Filters rows                     Filters groups
  Before GROUP BY                  After GROUP BY
  Cannot use aggregate functions   Can use aggregate functions

### SQL Execution Order

1.  FROM
2.  WHERE
3.  GROUP BY
4.  HAVING
5.  SELECT
6.  ORDER BY
7.  LIMIT

### Frequently Asked Interview Questions

#### COUNT(\*) vs COUNT(column)

-   `COUNT(*)` counts every row.
-   `COUNT(column)` counts only non-NULL values.
-   `COUNT(DISTINCT column)` counts unique non-NULL values.

#### Why aggregate functions cannot be used in WHERE?

Because `WHERE` executes before `GROUP BY` and aggregate calculations.

#### Can HAVING be used without GROUP BY?

Yes. The whole result set is treated as a single group.

#### Why every selected column must be in GROUP BY?

Every selected column must either: - be part of `GROUP BY`, or - be
wrapped inside an aggregate function.

Otherwise SQL cannot determine which value to return.

------------------------------------------------------------------------

# Practice Assignments

## Assignment 1 (Basic)

1.  Find employees earning more than 60000.
2.  Find employees whose names start with 'J'.
3.  Find employees whose names contain 'a'.
4.  Find employees between age 25 and 35.
5.  Find the second highest salary.
6.  Find employees in IT or HR.
7.  Display top 3 highest-paid employees.
8.  Find employees whose names have exactly 5 letters.
9.  Find employees not earning between 50000 and 70000.
10. Find employees with the minimum salary.

------------------------------------------------------------------------

## Assignment 2 (Aggregate Functions)

1.  Total number of employees.
2.  Total salary of all employees.
3.  Average salary.
4.  Highest salary in each department.
5.  Minimum salary in each department.
6.  Count employees in each department.
7.  Average age in each department.
8.  Departments having more than 2 employees.
9.  Departments with average salary greater than 60000.
10. Department having the highest total salary.

------------------------------------------------------------------------

# Interview Cheat Sheet

✅ WHERE → Filters rows

✅ GROUP BY → Creates groups

✅ HAVING → Filters groups

✅ COUNT(\*) → Counts all rows

✅ COUNT(column) → Counts non-NULL values

✅ COUNT(DISTINCT column) → Counts unique values

✅ Aggregate functions: - COUNT - SUM - AVG - MIN - MAX

### Remember SQL Execution Order

FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT

------------------------------------------------------------------------

## Next Chapter

Chapter 3: JOINS - INNER JOIN - LEFT JOIN - RIGHT JOIN - FULL OUTER
JOIN - SELF JOIN - CROSS JOIN - 10 Interview Questions - 10 Practice
Problems
