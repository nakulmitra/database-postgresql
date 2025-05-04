# Understanding LEFT JOIN in SQL
The `LEFT JOIN` is one of the fundamental SQL join operations that combines rows from two tables based on a related column. It ensures that all rows from the **left table** (the first table in the query) are included in the result set, even if there’s no matching row in the **right table**. If no match is found in the right table, the columns from the right table are filled with `NULL` values.

[![](https://markdown-videos-api.jorgenkh.no/youtube/za4XygGkw-s)](https://youtu.be/za4XygGkw-s)

## Key Concepts of LEFT JOIN

### 1. Basic Syntax
The syntax for a `LEFT JOIN` query is as follows:
```sql
SELECT column1, column2, ...
FROM table1
LEFT JOIN table2
ON table1.common_column = table2.common_column;
```
- **`SELECT`**: Specifies the columns you want to retrieve from the tables.
- **`FROM table1`**: Refers to the first (left) table in the query.
- **`LEFT JOIN table2`**: Combines rows from the left table with the right table, ensuring all rows from the left table are included in the result.
- **`ON table1.common_column = table2.common_column`**: Specifies the condition for matching rows between the two tables.

### 2. Behavior of LEFT JOIN
- A `LEFT JOIN` includes all rows from the **left table**, regardless of whether there is a matching row in the **right table**.
- If no match is found in the **right table**, the columns from the **right table** will contain `NULL` values.

### Example Scenario

Consider the following two tables:

#### Table: employees
| emp_id | emp_name  | dept_id |
|--------|-----------|---------|
| 1      | Aman     | 101     |
| 2      | Shreya       | 102     |
| 3      | Janet   | NULL    |
| 4      | Harshada     | 103     |

#### Table: departments
| dept_id | dept_name   |
|---------|-------------|
| 101     | HR          |
| 102     | Sales       |
| 104     | IT          |

#### LEFT JOIN Query
```sql
SELECT employees.emp_id, employees.emp_name, departments.dept_name
FROM employees
LEFT JOIN departments
ON employees.dept_id = departments.dept_id;
```

#### Result:
| emp_id | emp_name  | dept_name   |
|--------|-----------|-------------|
| 1      | Aman     | HR          |
| 2      | Shreya       | Sales       |
| 3      | Janet   | NULL        |
| 4      | Harshada     | NULL        |

**Explanation:**
1. **Aman and Shreya**: Their `dept_id` values (101 and 102) match rows in the `departments` table, so the corresponding `dept_name` values ("HR" and "Sales") are displayed.
2. **Janet and Harshada**: They appear in the result set because `LEFT JOIN` includes all rows from the `employees` table, but their `dept_name` values are `NULL` since there's no match in the `departments` table.

### Visual Representation
To help visualize a `LEFT JOIN`, imagine two circles:
- **Left Circle**: Represents rows from the left table (`employees`).
- **Right Circle**: Represents rows from the right table (`departments`).
The result of a `LEFT JOIN` includes:
1. The intersection (where the two circles overlap), representing matching rows.
2. All non-matching rows from the left circle (left table).

### Practical Use Cases

1. **Retrieving All Employees with Departments:**
   You may want to list all employees along with their department names, including those who aren't assigned to any department.

2. **Data Audits:**
   Use a `LEFT JOIN` to find rows in the left table that don't have corresponding entries in the right table. For example:
   ```sql
   SELECT employees.emp_id, employees.emp_name
   FROM employees
   LEFT JOIN departments
   ON employees.dept_id = departments.dept_id
   WHERE departments.dept_id IS NULL;
   ```
   **Purpose:** This query finds employees who are not assigned to any department.

3. **Filling Missing Data:**
   A `LEFT JOIN` is often used to combine primary data with supplementary data (e.g., employee details with optional performance data).

### **Key Notes**
1. **Comparison to INNER JOIN**:
   - While an `INNER JOIN` excludes non-matching rows, a `LEFT JOIN` ensures that all rows from the left table are included.
2. **Performance Considerations**:
   - `LEFT JOIN` queries can be slower than `INNER JOIN` for large datasets, as they require the database to retain all rows from the left table, including those with no match.
3. **NULL Handling**:
   - Be cautious when using `NULL` values in conditions or aggregations with `LEFT JOIN`.

### **Summary**
- **Definition**: A `LEFT JOIN` ensures all rows from the left table are included in the result set, with `NULL` values for non-matching rows from the right table.
- **Key Syntax**:
   ```sql
   SELECT columns
   FROM table1
   LEFT JOIN table2
   ON table1.common_column = table2.common_column;
   ```
- **Use Cases**: Displaying all records from a primary table (e.g., employees) while optionally joining with related data (e.g., departments).
- **Benefits**: Retains all rows from the left table, making it suitable for tasks like data audits or merging supplementary data.

### **Best Practices**
1. Use `LEFT JOIN` judiciously, as it may return more rows than needed if not properly filtered.
2. Always handle `NULL` values in your queries, especially when performing comparisons or aggregations.
3. Optimize queries by indexing the join columns to improve performance.