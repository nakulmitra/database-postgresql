# Understanding WHERE and HAVING Clauses in SQL
The **WHERE** and **HAVING** clauses are essential components of SQL, used to filter data in different stages of query execution. Understanding the difference between the two is crucial for writing efficient and accurate SQL queries. Below is a comprehensive explanation of both clauses with examples, key differences, and common mistakes.

[![](https://markdown-videos-api.jorgenkh.no/youtube/IlSOpQycJmo)](https://youtu.be/IlSOpQycJmo)

## 1. Overview of WHERE and HAVING Clauses
* `WHERE Clause:` The **WHERE** clause is used to filter rows before any grouping or aggregation is performed. It applies conditions directly to individual rows of data in a table.

* `HAVING Clause:` The **HAVING** clause is used to filter grouped or aggregated results after the `GROUP BY` clause is applied. It operates on aggregated data, such as sums, averages, or counts.

## 2. WHERE Clause
### Purpose
The WHERE clause is used to specify conditions that individual rows must meet to be included in the result set.
### Syntax
```sql
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

### Example 1: Basic Filtering:
Let's filter employees with a salary greater than 50,000
```sql
SELECT emp_id, first_name, salary
FROM employees
WHERE salary > 50000;
```

### Example 2: Multiple Conditions Using AND/OR
To filter employees older than 30 and earning more than 50,000
```sql
SELECT emp_id, first_name, age, salary
FROM employees
WHERE age > 30 AND salary > 50000;
```

### Key Notes on WHERE Clause
* The WHERE clause cannot use aggregate functions like `SUM`, `AVG`, or `COUNT`.
* It works at the row level, filtering raw data before any grouping or aggregation.

## 3. HAVING Clause
### Purpose
The HAVING clause is used to filter aggregated data after the `GROUP BY` clause is applied.
### Syntax
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1
HAVING condition;
```

### Example 1: Filtering Aggregated Data
Find departments where the average salary is greater than 60,000
```sql
SELECT department_name, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_name
HAVING AVG(salary) > 60000;
```

### Example 2: Combining Aggregates with HAVING
Find departments where the total salary exceeds 150,000
```sql
SELECT department_name, SUM(salary) AS total_salary
FROM employees
GROUP BY department_name
HAVING SUM(salary) > 150000;
```

### Key Notes on HAVING Clause
* The HAVING clause must be used with grouped data.
* Aggregate functions like `SUM`, `AVG`, `COUNT`, etc., can only be used in the HAVING clause, not in the WHERE clause.
* It works at the group level, filtering data after it has been grouped.

## 4. Key Differences Between WHERE and HAVING Clauses
| Aspect | WHERE Clause | HAVING Clause |
| -------|------------|-----------|
|    Application Stage   |  Filters rows before grouping or aggregation.  |  Filters groups after grouping or aggregation.  |
|    Works On   |  Individual rows.  |  Aggregated data or groups.  |
|    Aggregate Functions   |  Not Allowed.  |  Allowed (e.g., SUM, AVG).  |
|    Use Case   |  Filtering raw data.  |  Filtering grouped data.  |

## 5. Combining WHERE and HAVING
In many queries, both WHERE and HAVING can be used together to filter data at different stages.
### Example: Combining WHERE and HAVING
Find departments where:
* Employees earning below 40,000 are excluded.
* Total salary for the remaining employees exceeds 150,000.
```sql
SELECT department_name, SUM(salary) AS total_salary
FROM employees
WHERE salary > 40000
GROUP BY department_name
HAVING SUM(salary) > 150000;
```
### Explanation:
* The `WHERE` clause filters out individual rows where salary ≤ 40,000.
* The `GROUP BY` clause groups the remaining rows by department.
* The `HAVING` clause filters departments where the total salary ≤ 150,000.

## 6. Common Mistakes to Avoid
### 1. Using Aggregate Functions in WHERE Clause
Aggregate functions like `SUM` or `AVG` cannot be used in the WHERE clause. This will throw an error:
```sql
SELECT department_name
FROM employees
WHERE SUM(salary) > 150000; -- Incorrect
```

Instead, use the HAVING clause:
```sql
SELECT department_name, SUM(salary) AS total_salary
FROM employees
GROUP BY department_name
HAVING SUM(salary) > 150000;
```

### 2. Using HAVING Without GROUP BY
Although HAVING can work without GROUP BY, it's rarely meaningful:
```sql
SELECT sum(salary)
FROM employees
HAVING sum(salary) > 50000; -- Technically valid but not recommended
```

## 7. Best Practices
* Use WHERE for filtering raw data before grouping or aggregation.
* Use HAVING for filtering aggregated results or grouped data.
* Always combine WHERE and HAVING for complex queries to optimize performance.
* Double-check your conditions in WHERE and HAVING to avoid unintended results.

## 8. Summary
* The **WHERE** clause filters rows before grouping or aggregation, working on raw data.
* The **HAVING** clause filters grouped or aggregated data after applying the GROUP BY clause.
* Combine WHERE and HAVING for more complex queries that involve both row-level and group-level filtering.
* Avoid common pitfalls like using aggregate functions in WHERE or misusing HAVING without grouping.