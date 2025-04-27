# Subqueries in SQL
Welcome to this detailed guide on **Subqueries in SQL**!
This document explains what subqueries are, their types, practical examples, efficiency tips, common mistakes, and best practices.

## What is a Subquery?
A **subquery** is a query embedded inside another SQL query.
It is used to perform operations that require multiple steps or intermediary results before delivering the final output.

You can think of a subquery as a **"query inside a query"** that provides data to the main (outer) query.

### Where can Subqueries be Used?
- In `WHERE` clauses (to filter rows)
- In `FROM` clauses (as a derived table)
- In `SELECT` lists (to calculate values)
- In `HAVING` clauses (for filtering groups)

## Types of Subqueries
Subqueries can be categorized based on:
1. The number of rows returned
2. Whether they are dependent on the outer query

### 1. Based on the Result:

| Type | Description | Example |
|:----|:------------|:--------|
| Single-row Subquery | Returns only one value | `WHERE salary > (SELECT AVG(salary) FROM employees);` |
| Multi-row Subquery | Returns multiple rows | `WHERE dept_id IN (SELECT dept_id FROM employees WHERE salary > 30000);` |

### 2. Based on Dependency:

| Type | Description | Example |
|:----|:------------|:--------|
| Correlated Subquery | Depends on the outer query for its value | `WHERE salary > (SELECT AVG(salary) FROM employees WHERE e1.dept_id = employees.dept_id);` |
| Uncorrelated Subquery | Independent of the outer query | `WHERE dept_id IN (SELECT dept_id FROM employees);` |

## Practical Examples

### Example 1: Single-row Subquery
Find employees earning more than the company's average salary:

```
SELECT employee_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### Example 2: Multi-row Subquery
Find department names where employees earn more than 30K:

```
SELECT dept_name
FROM departments
WHERE dept_id IN (
    SELECT dept_id
    FROM employees
    WHERE salary > 30000
);
```

### Example 3: Correlated Subquery
List employees who earn more than the average salary in their department:

```
SELECT employee_name, department_id, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e1.department_id = e2.department_id
);
```

### Example 4: Subquery with Aggregates
Find the department with the highest total salary:

```
SELECT department_id, total_salary
FROM (
    SELECT department_id, SUM(salary) AS total_salary
    FROM employees
    GROUP BY department_id
) AS department_salaries
WHERE total_salary = (
    SELECT MAX(total_salary)
    FROM (
        SELECT SUM(salary) AS total_salary
        FROM employees
        GROUP BY department_id
    ) AS department_totals
);
```

## Writing Efficient Subqueries

Subqueries are powerful but can become a performance bottleneck if not handled properly. Here are key tips:

| Best Practice | Explanation |
|:-------------|:------------|
| **Avoid Repetition** | If using the same subquery multiple times, consider using a Common Table Expression (CTE) or temporary table. |
| **Filter Early** | Apply `WHERE` clauses inside subqueries to reduce unnecessary rows early. |
| **Use Indexes** | Ensure the columns used in subqueries are indexed. |
| **Simplify with Joins** | In many cases, a `JOIN` might perform better than a correlated subquery. |
| **Be Cautious with Correlated Subqueries** | They are executed for each row of the outer query, which can be expensive. |

## Common Mistakes to Avoid

| Mistakes | Why It's Bad |
|:-------|:-------------|
| **Overusing Correlated Subqueries** | Causes row-by-row execution, leading to slow performance on large tables. |
| **Complex Nested Subqueries** | Makes the query hard to debug and maintain. |
| **Ignoring Indexes** | Results in full table scans inside subqueries, degrading performance. |
| **Not Analyzing Execution Plans** | You should always review the query plan using tools like `EXPLAIN ANALYZE`. |

## Summary
* **Subqueries** are powerful and flexible for solving complex SQL problems.
* Use **single-row**, **multi-row**, **correlated**, and **uncorrelated** subqueries appropriately based on your need.
* Always **optimize subqueries** for performance: use filters, indexes, and CTEs when necessary.
* Be mindful of **common pitfalls** like overusing correlated subqueries and ignoring indexing.