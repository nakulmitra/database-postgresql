# RIGHT JOIN in SQL
In SQL, **RIGHT JOIN** (also known as **RIGHT OUTER JOIN**) is used to combine rows from two tables, keeping all the rows from the **right table** and matching rows from the **left table**. If there is no match, NULL values are returned for columns from the left table.

The **RIGHT JOIN** operation ensures that all records from the **right table** are returned, regardless of whether a match exists in the **left table**.

## Basic Syntax of RIGHT JOIN
```
SELECT column1, column2, ...
FROM table1
RIGHT JOIN table2
ON table1.common_column = table2.common_column;
```

- **SELECT column1, column2, ...**: Specifies the columns you want to retrieve from the tables.
- **FROM table1**: Specifies the first table (also called the **left table**).
- **RIGHT JOIN table2**: Specifies the second table (also called the **right table**) and indicates that all rows from this table should be included.
- **ON table1.common_column = table2.common_column**: Defines the condition for the join, specifying how the tables are related (typically through a common column).

## How RIGHT JOIN Works
- **RIGHT JOIN** ensures that every row from the **right table** appears in the result set. If a matching row exists in the **left table**, the corresponding data from both tables is shown. If there's no match, the columns from the left table will contain **NULL** values.
- The main idea is that the **right table** is given priority, so even if there's no match for a row from the left table, that row from the right table will still be included in the output.

### **Example**
Consider the following two tables:
1. **employees**

| emp_id | emp_name | dept_id |
|--------|----------|---------|
| 1      | Aman    | 101     |
| 2      | Shreya      | 102     |
| 3      | Janet  | 103     |

2. **departments**

| dept_id | dept_name |
|---------|-----------|
| 101     | HR        |
| 102     | Sales     |
| 104     | IT        |

Now, if we perform a **RIGHT JOIN** on these tables, where we want to retrieve all the department names along with the employees assigned to them, the query would look like this:

```
SELECT employees.emp_id, employees.emp_name, departments.dept_name
FROM employees
RIGHT JOIN departments
ON employees.dept_id = departments.dept_id;
```

### **Output**

| emp_id | emp_name | dept_name |
|--------|----------|-----------|
| 1      | Alice    | HR        |
| 2      | Shreya      | Sales     |
| NULL   | NULL     | IT        |

### Explanation:
- For departments **HR** and **Sales**, we have matching rows in the **employees** table, so we get the employee details (emp_id, emp_name) alongside the department.
- For the **IT** department, there's no employee in the **employees** table with a matching `dept_id`, so the result for the **IT** department includes `NULL` values for `emp_id` and `emp_name`.

## Key Points About RIGHT JOIN

1. **Prioritizes the right table**: All records from the right table are included in the result set, regardless of whether there's a matching record in the left table.
2. **NULL values**: If no matching record exists in the left table, the result set will contain `NULL` for the corresponding columns from the left table.
3. **Contrast with LEFT JOIN**: In **LEFT JOIN**, all records from the left table are returned, while in **RIGHT JOIN**, all records from the right table are returned. Both joins behave similarly but focus on different tables.
4. **Application in real-world scenarios**: RIGHT JOIN is especially useful when you need to ensure all records from the **right table** appear in your query results, even if the left table has no corresponding data.

## Practical Use Case
In real-world applications, you might use RIGHT JOIN when you want to retrieve data from a primary table, and you want to ensure you show records even if there's no associated data in a secondary table. For example:
- Reporting systems where you need to show all departments, even if no employees are assigned.
- Aggregating data where some categories might not have matching entries in a related table.

## Summary
- **RIGHT JOIN** returns all rows from the **right table** and matching rows from the **left table**. If there's no match, NULL values are filled in for columns from the left table.
- It's commonly used when you want to ensure the **right table**'s data is fully represented in your result set.
- It's the reverse of **LEFT JOIN**, which prioritizes the left table.

By understanding and applying **RIGHT JOIN**, you can handle various scenarios where you need to combine data from multiple tables with specific priorities.