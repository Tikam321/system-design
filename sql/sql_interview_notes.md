# SQL Interview Handbook
## Chapters 1 & 2

---

# Chapter 1 - Basic SQL

## What is SQL?

SQL (Structured Query Language) is used to store, retrieve, update and delete data from relational databases.

Common commands:

- SELECT
- INSERT
- UPDATE
- DELETE
- CREATE
- ALTER
- DROP

---

# Sample Table

| id | name | salary | department | age |
|----|------|--------|------------|-----|
|1|John|50000|IT|25|
|2|David|60000|HR|30|
|3|Emma|70000|IT|35|
|4|Lisa|45000|Finance|28|
|5|Mike|90000|IT|40|

---

# SELECT

Retrieve data.

```sql
SELECT *
FROM Employee;
```

Retrieve specific columns

```sql
SELECT name,salary
FROM Employee;
```

---

# WHERE

Filters rows.

```sql
SELECT *
FROM Employee
WHERE salary>50000;
```

Operators

```
=
!=
<>
>
<
>=
<=
```

---

# BETWEEN

Inclusive.

```sql
SELECT *
FROM Employee
WHERE salary BETWEEN 50000 AND 70000;
```

Equivalent to

```sql
salary>=50000
AND
salary<=70000
```

---

# IN

```sql
SELECT *
FROM Employee
WHERE department IN ('IT','HR');
```

---

# DISTINCT

Returns unique values.

```sql
SELECT DISTINCT department
FROM Employee;
```

Output

IT

HR

Finance

---

# ORDER BY

Ascending

```sql
ORDER BY salary ASC;
```

Descending

```sql
ORDER BY salary DESC;
```

Multiple columns

```sql
ORDER BY age DESC,name ASC;
```

---

# LIMIT

Top N records.

```sql
SELECT *
FROM Employee
ORDER BY salary DESC
LIMIT 3;
```

---

# LIKE

Starts with J

```sql
LIKE 'J%'
```

Ends with n

```sql
LIKE '%n'
```

Contains i

```sql
LIKE '%i%'
```

Exactly 5 letters

```sql
LIKE '_____'
```

One character

```
_
```

Multiple characters

```
%
```

---

# Subquery Example

Employees earning above average salary.

```sql
SELECT *
FROM Employee
WHERE salary>
(
SELECT AVG(salary)
FROM Employee
);
```

---

# Assignment 1

## Question 1

Find employees earning more than 60000.

```sql
SELECT *
FROM Employee
WHERE salary>60000;
```

---

## Question 2

Names starting with J

```sql
SELECT *
FROM Employee
WHERE name LIKE 'J%';
```

---

## Question 3

Names containing a

```sql
SELECT *
FROM Employee
WHERE name LIKE '%a%';
```

---

## Question 4

Employees between age 25 and 35

```sql
SELECT *
FROM Employee
WHERE age BETWEEN 25 AND 35;
```

---

## Question 5

Second highest salary

```sql
SELECT DISTINCT salary
FROM Employee
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

Alternative

```sql
SELECT MAX(salary)
FROM Employee
WHERE salary<
(
SELECT MAX(salary)
FROM Employee
);
```

---

## Question 6

Employees in IT or HR

```sql
SELECT *
FROM Employee
WHERE department IN ('IT','HR');
```

---

## Question 7

Top three highest paid employees

```sql
SELECT *
FROM Employee
ORDER BY salary DESC
LIMIT 3;
```

---

## Question 8

Exactly five letter names

```sql
SELECT *
FROM Employee
WHERE name LIKE '_____';
```

---

## Question 9

Salary NOT BETWEEN 50000 and 70000

```sql
SELECT *
FROM Employee
WHERE salary NOT BETWEEN 50000 AND 70000;
```

---

## Question 10

Employee with minimum salary

```sql
SELECT *
FROM Employee
WHERE salary=
(
SELECT MIN(salary)
FROM Employee
);
```

---

# Chapter 1 Interview Questions

### Difference between WHERE and HAVING?

WHERE filters rows.

HAVING filters groups after aggregation.

---

### Difference between LIKE '%' and '_'?

% → Any number of characters

_ → Exactly one character

---

### Difference between DISTINCT and GROUP BY?

DISTINCT removes duplicate rows.

GROUP BY groups rows to perform aggregate functions.

---

### LIMIT vs TOP?

LIMIT → MySQL/PostgreSQL

TOP → SQL Server

---

# Chapter 2 - Aggregate Functions

Aggregate functions operate on multiple rows and return a single value.

---

# COUNT

Counts rows.

```sql
SELECT COUNT(*)
FROM Employee;
```

---

# COUNT(column)

Counts only NON NULL values.

```sql
COUNT(name)
```

---

# COUNT(DISTINCT)

Counts unique values.

```sql
COUNT(DISTINCT department)
```

---

# SUM

```sql
SELECT SUM(salary)
FROM Employee;
```

---

# AVG

```sql
SELECT AVG(salary)
FROM Employee;
```

---

# MIN

```sql
SELECT MIN(salary)
FROM Employee;
```

---

# MAX

```sql
SELECT MAX(salary)
FROM Employee;
```

---

# GROUP BY

Creates groups.

```sql
SELECT department,
AVG(salary)
FROM Employee
GROUP BY department;
```

---

# HAVING

Filters groups.

```sql
SELECT department,
AVG(salary)
FROM Employee
GROUP BY department
HAVING AVG(salary)>60000;
```

---

# SQL Execution Order

1 FROM

2 WHERE

3 GROUP BY

4 HAVING

5 SELECT

6 ORDER BY

7 LIMIT

This is one of the most asked interview questions.

---

# Assignment 2

## Question 1

Total employees

```sql
SELECT COUNT(*)
FROM Employee;
```

---

## Question 2

Total salary

```sql
SELECT SUM(salary)
FROM Employee;
```

---

## Question 3

Average salary

```sql
SELECT AVG(salary)
FROM Employee;
```

---

## Question 4

Highest salary per department

```sql
SELECT department,
MAX(salary)
FROM Employee
GROUP BY department;
```

---

## Question 5

Minimum salary per department

```sql
SELECT department,
MIN(salary)
FROM Employee
GROUP BY department;
```

---

## Question 6

Employee count per department

```sql
SELECT department,
COUNT(*)
FROM Employee
GROUP BY department;
```

---

## Question 7

Average age per department

```sql
SELECT department,
AVG(age)
FROM Employee
GROUP BY department;
```

---

## Question 8

Departments having more than two employees

```sql
SELECT department,
COUNT(*)
FROM Employee
GROUP BY department
HAVING COUNT(*)>2;
```

---

## Question 9

Departments having average salary greater than 60000

```sql
SELECT department,
AVG(salary)
FROM Employee
GROUP BY department
HAVING AVG(salary)>60000;
```

---

## Question 10

Department having highest total salary

```sql
SELECT department,
SUM(salary) total_salary
FROM Employee
GROUP BY department
ORDER BY total_salary DESC
LIMIT 1;
```

---

# Most Important Interview Questions

### COUNT(*) vs COUNT(column)

COUNT(*) counts every row.

COUNT(column) ignores NULL values.

---

### COUNT(*) vs COUNT(DISTINCT)

COUNT(*) counts all rows.

COUNT(DISTINCT column) counts only unique non-NULL values.

---

### Can HAVING be used without GROUP BY?

Yes.

The entire table becomes one group.

Example

```sql
SELECT COUNT(*)
FROM Employee
HAVING COUNT(*)>0;
```

---

### Why aggregate functions cannot be used inside WHERE?

Because WHERE executes before GROUP BY and aggregate calculations.

Use HAVING instead.

---

### Why every selected column must be in GROUP BY?

Because SQL must know which value to return for each group.

Every selected column should either

- be in GROUP BY

OR

- be aggregated.

---

### Which executes first?

```
FROM
↓

WHERE
↓

GROUP BY
↓

HAVING
↓

SELECT
↓

ORDER BY
↓

LIMIT
```

---

# Chapter 1 & 2 Cheat Sheet

✔ SELECT → Retrieve data

✔ WHERE → Filter rows

✔ GROUP BY → Create groups

✔ HAVING → Filter groups

✔ ORDER BY → Sort

✔ LIMIT → Restrict rows

✔ DISTINCT → Remove duplicates

✔ COUNT(*) → Count all rows

✔ COUNT(column) → Count non-NULL values

✔ COUNT(DISTINCT) → Count unique values

✔ SUM → Total

✔ AVG → Average

✔ MIN → Minimum

✔ MAX → Maximum

✔ LIKE '%' → Multiple characters

✔ LIKE '_' → Single character

✔ SQL Execution Order

FROM

↓

WHERE

↓

GROUP BY

↓

HAVING

↓

SELECT

↓

ORDER BY

↓

LIMIT

---

# Next Chapter

Chapter 3

SQL JOINS

- INNER JOIN
- LEFT JOIN
- RIGHT JOIN
- FULL OUTER JOIN
- SELF JOIN
- CROSS JOIN
- 10 Assignments
- 10 Solutions
- Interview Questions
- Common Mistakes
