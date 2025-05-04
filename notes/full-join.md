# Understanding FULL JOIN in SQL
SQL `FULL JOIN` (also known as **FULL OUTER JOIN**) is used to retrieve all records from both tables, whether there is a match or not. If a record has no match in one of the tables, `NULL` values are returned for the missing columns.

FULL JOIN is particularly useful when we want to retain all data from both tables while merging related records based on a common column.

[![](https://markdown-videos-api.jorgenkh.no/youtube/UiGIHmY65tM)](https://youtu.be/UiGIHmY65tM)

## Syntax of FULL JOIN
```sql
SELECT column1, column2, ...
FROM table1
FULL JOIN table2
ON table1.common_column = table2.common_column;
```
- `table1` and `table2`: The tables involved in the join.
- `ON table1.common_column = table2.common_column`: Specifies the condition on which the join is performed.

## Example of FULL JOIN
Consider two tables:

### **Employees Table**
| emp_id | emp_name | dept_id |
|--------|---------|---------|
| 1      | Aman   | 101     |
| 2      | Shreya     | 102     |
| 3      | Janet | NULL    |
| 4      | Harshada   | NULL    |

### **Departments Table**
| dept_id | dept_name  |
|---------|-----------|
| 101     | HR        |
| 102     | Sales     |
| 103     | IT        |

Now, we perform a **FULL JOIN** between `employees` and `departments`:

```sql
SELECT employees.emp_id, employees.emp_name, departments.dept_name
FROM employees
FULL JOIN departments
ON employees.dept_id = departments.dept_id;
```

### **Result:**
| emp_id | emp_name | dept_name |
|--------|---------|-----------|
| 1      | Aman   | HR        |
| 2      | Shreya     | Sales     |
| 3      | Janet | NULL      |
| 4      | Harshada   | NULL      |
| NULL   | NULL    | IT        |

### **Explanation:**
1. **Matched Rows:** `Aman` and `Shreya` have matching department IDs (`101` and `102`), so their respective `dept_name` is displayed.
2. **Employees Without Departments:** `Janet` and `Harshada` have `NULL` as their `dept_id`, so `dept_name` is `NULL`.
3. **Departments Without Employees:** The `IT` department (`dept_id = 103`) has no employees, so `emp_id` and `emp_name` are `NULL`.

## Visual Representation of FULL JOIN
A **FULL JOIN** returns all records from both tables:
- Matching rows are combined.
- Non-matching rows from both tables are retained with NULL values.

## Comparison with Other Joins
| JOIN Type    | Result |
|-------------|--------|
| **INNER JOIN**  | Returns only matching rows. |
| **LEFT JOIN**   | Returns all rows from the left table and matching rows from the right. Unmatched rows from the right table return `NULL`. |
| **RIGHT JOIN**  | Returns all rows from the right table and matching rows from the left. Unmatched rows from the left table return `NULL`. |
| **FULL JOIN** | Returns all rows from both tables, with `NULL` for unmatched rows. |

## When to Use FULL JOIN?
Use `FULL JOIN` when:
- We need to retrieve **all data from two related tables**, even if there are no exact matches.
- We want to analyze missing relationships between two datasets.
- We want to combine **LEFT JOIN** and **RIGHT JOIN** behavior in one query.

Avoid `FULL JOIN` if:
- We only need matching rows (**INNER JOIN** is better).
- We need all rows from just one table (**LEFT JOIN** or **RIGHT JOIN** is more efficient).

## Handling NULL Values in FULL JOIN
Since `FULL JOIN` can return `NULL` values for unmatched records, we can use **COALESCE()** to replace NULLs with default values.

```sql
SELECT 
    COALESCE(employees.emp_id, 'No Employee') AS emp_id,
    COALESCE(employees.emp_name, 'No Name') AS emp_name,
    COALESCE(departments.dept_name, 'No Department') AS dept_name
FROM employees
FULL JOIN departments
ON employees.dept_id = departments.dept_id;
```

**Output:**
| emp_id       | emp_name   | dept_name       |
|-------------|-----------|----------------|
| 1           | Aman     | HR             |
| 2           | Shreya       | Sales          |
| 3           | Janet   | No Department  |
| 4           | Harshada     | No Department  |
| No Employee | No Name   | IT             |

## Performance Considerations
`FULL JOIN` can be slow on **large datasets** because:
- It needs to scan **both tables entirely**.
- It creates a **Cartesian-like merge**, which can be expensive.
- Indexing **common columns** can **improve performance**.

**Alternative Approaches:**
- If we don't need unmatched rows from **both** tables, consider `LEFT JOIN` or `RIGHT JOIN`.
- If we need **only matching rows**, use `INNER JOIN`.

## Common Errors & Solutions
### 1. FULL JOIN Not Supported in MySQL
MySQL **does not support FULL JOIN** directly. Instead, we can use `UNION` of `LEFT JOIN` and `RIGHT JOIN`:

```sql
SELECT employees.emp_id, employees.emp_name, departments.dept_name
FROM employees
LEFT JOIN departments
ON employees.dept_id = departments.dept_id

UNION

SELECT employees.emp_id, employees.emp_name, departments.dept_name
FROM employees
RIGHT JOIN departments
ON employees.dept_id = departments.dept_id;
```

### 2. NULL Values in JOIN Conditions
If the common column contains `NULL`, FULL JOIN **will not match those rows**. To handle this:

```sql
SELECT employees.emp_id, employees.emp_name, departments.dept_name
FROM employees
FULL JOIN departments
ON COALESCE(employees.dept_id, -1) = COALESCE(departments.dept_id, -1);
```

## Conclusion
* **FULL JOIN** retrieves all records from both tables.  
* It combines **LEFT JOIN** and **RIGHT JOIN** results.  
* NULL values appear where there's no match.  
* It's useful for **data reconciliation** and **relationship analysis**.
* Performance optimization is necessary for **large datasets**.