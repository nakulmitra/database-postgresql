## Understanding INNER JOIN in SQL
An **INNER JOIN** is one of the most commonly used types of JOINs in SQL. It allows you to retrieve rows from two or more tables where a specified condition is met. INNER JOIN is essential for combining data from relational tables and is widely used in data analysis, reporting, and database management.

[![](https://markdown-videos-api.jorgenkh.no/youtube/WQW_bGatSzg)](https://youtu.be/WQW_bGatSzg)

## 1. What Is an INNER JOIN?
An **INNER JOIN** returns rows from multiple tables where the join condition is satisfied. Rows that do not meet the condition are excluded from the result set. This type of JOIN is commonly used when you need to extract related data from multiple tables in a relational database.

### Key Points About INNER JOIN:
- Combines rows based on a related column between tables.
- Returns only the matching rows.
- Non-matching rows are excluded.

## 2. Basic Syntax of INNER JOIN
```
SELECT column1, column2, ...
FROM table1
INNER JOIN table2
ON table1.common_column = table2.common_column;
```

### Explanation of Syntax:
- `SELECT`: Specifies the columns to include in the result.
- `FROM table1`: The first table in the query.
- `INNER JOIN table2`: Combines rows from `table1` and `table2`.
- `ON table1.common_column = table2.common_column`: Specifies the condition for matching rows.

## 3. Example Tables
Let's take two tables, **employees** and **departments**, as an example:  

### Table: employees
| emp_id | emp_name | dept_id |
|--------|----------|---------|
| 1      | Aman    | 101     |
| 2      | Shreya      | 102     |
| 3      | Janet  | NULL    |
| 4      | Harshada    | 103     |

### Table: departments
| dept_id | dept_name |
|---------|-----------|
| 101     | HR        |
| 102     | Sales     |
| 104     | IT        |

## 4. INNER JOIN Query Example
```
SELECT employees.emp_id, employees.emp_name, departments.dept_name
FROM employees
INNER JOIN departments
ON employees.dept_id = departments.dept_id;
```

### Explanation of the Query:
- Combines rows from the `employees` table and the `departments` table.
- The **ON** clause ensures that only rows where `employees.dept_id` matches `departments.dept_id` are included in the result.
- Non-matching rows (e.g., rows with `NULL` values in `dept_id`) are excluded.

### Query Output:
| emp_id | emp_name | dept_name |
|--------|----------|-----------|
| 1      | Aman    | HR        |
| 2      | Shreya      | Sales     |

### Why This Output?
- **Aman**: Her `dept_id` (101) matches the `dept_id` in the `departments` table.
- **Shreya**: His `dept_id` (102) also has a match.
- **Janet and Harshada**: They are excluded because their `dept_id` values either don't match or are `NULL`.

## 5. Visual Representation of INNER JOIN
**Venn Diagram Explanation:**  
Imagine two overlapping circles:
- **Left Circle**: Represents rows from the `employees` table.
- **Right Circle**: Represents rows from the `departments` table.
- The **overlapping area**: Represents rows where `dept_id` matches in both tables.

## 6. Use Cases of INNER JOIN
1. **Combining Related Data**: Joining orders with customers to get customer details for each order.
2. **Filtering Relevant Data**: Fetching only employees with valid departments.
3. **Data Analysis**: Analyzing sales data by joining transactions with product details.

## 7. Key Considerations
- **Performance**: Ensure that the columns used in the **ON** clause are indexed for faster query execution.
- **Data Integrity**: INNER JOIN only works correctly when there is a well-defined relationship between the tables.
- **NULL Handling**: Rows with `NULL` values in the join column are excluded because `NULL` cannot equal any value.

## 8. Summary
- **INNER JOIN** retrieves rows that satisfy the join condition.
- It is ideal for combining related data from multiple tables.
- Rows that don't meet the condition are excluded.
- By mastering INNER JOIN, you can handle complex queries and unlock the full potential of relational databases.

## Example Code Snippet
Here's the complete example for reference:
```
-- Query to retrieve employees with their department names
SELECT employees.emp_id, employees.emp_name, departments.dept_name
FROM employees
INNER JOIN departments
ON employees.dept_id = departments.dept_id;
```