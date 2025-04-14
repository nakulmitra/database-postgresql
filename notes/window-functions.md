# Introduction
Window functions in SQL allow us to perform calculations across a group of related rows while still keeping each row intact. Unlike aggregate functions, which collapse rows into a single summary value, window functions retain all rows while computing results across a specified window of data.

This guide explains window functions in detail, covering **ROW_NUMBER(), RANK(), DENSE_RANK(), LAG(), LEAD(), SUM(), and AVG()** with practical examples. Additionally, we'll solve **two common SQL interview questions** using window functions.

[![](https://markdown-videos-api.jorgenkh.no/youtube/gMymeinwF8A)](https://youtu.be/gMymeinwF8A)

## Difference Between Aggregate and Window Functions

| Feature                 | Aggregate Functions (SUM, AVG, etc.) | Window Functions (ROW_NUMBER, RANK, etc.) |
|-------------------------|-------------------------------------|------------------------------------------|
| **Output**              | Returns a single value for a group | Returns a value for each row            |
| **Collapses Rows?**     | Yes, groups multiple rows into one | No, keeps all rows                      |
| **Uses GROUP BY?**      | Yes                                 | No (Uses OVER() instead)                |
| **Example**            | `SELECT dept_id, SUM(salary) FROM employees GROUP BY dept_id;` | `SELECT dept_id, salary, SUM(salary) OVER (PARTITION BY dept_id) FROM employees;` |

## Syntax of Window Functions
Window functions use the `OVER()` clause, which can have:

* **PARTITION BY** - Divides rows into groups (optional).
* **ORDER BY** - Specifies order within each partition (optional).

### Basic Syntax:
```
SELECT column_name, 
       window_function() OVER (PARTITION BY column ORDER BY column) AS result
FROM table_name;
```

## Important Window Functions in SQL

### ROW_NUMBER() - Assigns a Unique Row Number
The `ROW_NUMBER()` function assigns a unique sequential number to each row within a partition.

#### Example: Assigning Row Numbers to Employees by Department
```
SELECT e.first_name, d.dept_name, e.salary,
       ROW_NUMBER() OVER (PARTITION BY d.dept_name ORDER BY e.salary DESC) AS row_num
FROM public.employees e 
JOIN public.departments d ON e.dept_id = d.dept_id
ORDER BY d.dept_name, e.salary DESC;
```

**Explanation:**
- `PARTITION BY d.dept_name` ensures that row numbering is applied separately for each department.
- `ORDER BY e.salary DESC` assigns the highest salary a row number of 1 within each department.
- Even if two employees have the same salary, they get different row numbers, since `ROW_NUMBER()` does not handle ties.

### RANK() - Assigns Ranking with Gaps
The `RANK()` function assigns a ranking to each row, but identical values receive the same rank, and gaps appear in the ranking sequence.

#### Example: Ranking Employees by Salary
```
SELECT e.first_name, d.dept_name, e.salary,
       RANK() OVER (PARTITION BY d.dept_name ORDER BY e.salary DESC) AS rank
FROM public.employees e 
JOIN public.departments d ON e.dept_id = d.dept_id
ORDER BY d.dept_name, e.salary DESC;
```

**Explanation:**
- Identical salaries share the same rank.
- If two employees have the same salary at rank 2, the next rank is **4** (skips rank 3).

### DENSE_RANK() - Assigns Ranking Without Gaps
`DENSE_RANK()` works like `RANK()`, but without skipping ranks.

#### Example: Dense Ranking Employees by Salary
```
SELECT e.first_name, d.dept_name, e.salary,
       DENSE_RANK() OVER (PARTITION BY d.dept_name ORDER BY e.salary DESC) AS dense_rank
FROM public.employees e 
JOIN public.departments d ON e.dept_id = d.dept_id
ORDER BY d.dept_name, e.salary DESC;
```

**Explanation:**
- If two employees have the same salary at rank 2, the next rank is **3** (no skipping).

### LAG() - Fetches Previous Row Value
The `LAG()` function allows us to access the value of a column from the **previous row**.

#### Example: Fetching Previous Employee's Salary
```
SELECT e.first_name, d.dept_name, e.salary,
       LAG(e.salary) OVER (PARTITION BY d.dept_name ORDER BY e.salary DESC) AS previous_salary
FROM public.employees e 
JOIN public.departments d ON e.dept_id = d.dept_id
ORDER BY d.dept_name, e.salary DESC;
```

**Explanation:**
- The **first row in each department will have NULL** since there is no previous row.

### LEAD() - Fetches Next Row Value
The `LEAD()` function retrieves the **next row’s value** based on the ordering criteria.

#### Example: Fetching Next Employee's Salary
```
SELECT e.first_name, d.dept_name, e.salary,
       LEAD(e.salary) OVER (PARTITION BY d.dept_name ORDER BY e.salary DESC) AS next_salary
FROM public.employees e 
JOIN public.departments d ON e.dept_id = d.dept_id
ORDER BY d.dept_name, e.salary DESC;
```

**Explanation:**
- The **last row in each partition will have NULL** since there is no next row.

### SUM() - Running Total (Cumulative Sum)
The `SUM()` function computes a running total of a column.

#### Example: Calculating Running Salary Total
```
SELECT e.first_name, d.dept_name, e.salary,
       SUM(e.salary) OVER (PARTITION BY d.dept_name ORDER BY e.salary DESC) AS running_total
FROM public.employees e 
JOIN public.departments d ON e.dept_id = d.dept_id
ORDER BY d.dept_name, e.salary DESC;
```

**Explanation:**
- Each row's `running_total` includes all previous salaries within the department.

## Real-World Use Cases of Window Functions

### Interview Question 1: Find the Top 3 Highest Salaries in Each Department
#### Solution Using ROW_NUMBER()
```
SELECT * FROM (
    SELECT e.first_name, d.dept_name, e.salary,
           ROW_NUMBER() OVER (PARTITION BY d.dept_name ORDER BY e.salary DESC) AS row_num
    FROM public.employees e 
    JOIN public.departments d ON e.dept_id = d.dept_id
) ranked WHERE row_num <= 3;
```
* **Use Case:** Finding the top 3 highest-paid employees per department.

### Interview Question 2: Find Salary Difference Between Current Employee and Previous Employee
#### Solution Using LAG()
```
SELECT e.first_name, d.dept_name, e.salary,
       salary - LAG(salary) OVER (PARTITION BY d.dept_name ORDER BY e.salary DESC) AS salary_difference
FROM public.employees e 
JOIN public.departments d ON e.dept_id = d.dept_id
ORDER BY d.dept_name, e.salary DESC;
```
* **Use Case:** Determining salary differences between consecutive employees.

## Summary
* **Window functions allow row-wise calculations without collapsing rows.**
* `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` are used for **ranking data**.
* `LAG()` and `LEAD()` fetch **previous and next row values**.
* `SUM()` and `AVG()` compute **cumulative and moving totals**.
* **Real-world applications include ranking employees, calculating salary differences, and finding top N records.**