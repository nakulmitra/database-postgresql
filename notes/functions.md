### Understanding Functions in PostgreSQL
In PostgreSQL, functions play a crucial role in database programming by allowing developers to create reusable logic that performs computations and returns values. Functions are similar to stored procedures but have key differences in terms of functionality and usage.

[![](https://markdown-videos-api.jorgenkh.no/youtube/0ox3f6IW5KM)](https://youtu.be/0ox3f6IW5KM)

## 1. Difference Between Functions and Stored Procedures

| Feature               | Functions | Stored Procedures |
|----------------------|-----------|------------------|
| **Purpose**          | Perform computations and return a value | Automate tasks and handle transactions |
| **Return Value**     | Must return a value (scalar, table, or JSON) | Do not return a value |
| **Transaction Control** | Cannot include COMMIT or ROLLBACK | Can include COMMIT or ROLLBACK |
| **Usage**           | Used for calculations, data transformations, and queries | Used for complex business logic and process automation |

### When to Use Functions?
* When a computation needs to return a single value or a result set.
* When a reusable logic needs to be embedded in queries.
* When transactions are not required within the logic.

## 2. Syntax of Functions in PostgreSQL

### Basic Syntax
```sql
CREATE FUNCTION function_name(param1 DATA_TYPE, param2 DATA_TYPE, ...)
RETURNS RETURN_TYPE
LANGUAGE plpgsql
AS $$
BEGIN
    -- Function logic
    RETURN some_value;
END;
$$;
```

### Example 1: Creating a Simple Function
Let's create a function to calculate an employee's **annual salary** based on their monthly salary.  
```sql
CREATE FUNCTION calculate_annual_salary(monthly_salary NUMERIC)
RETURNS NUMERIC
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN monthly_salary * 12;
END;
$$;
```

### Calling the Function
We can call this function using the `SELECT` statement:  
```sql
SELECT calculate_annual_salary(5000);
```
**Output**  
```
60000
```

## 3. Functions with Multiple Parameters
A function can take multiple parameters for more complex calculations.  

### Example 2: Function with Two Parameters (Calculating Bonus)
```sql
CREATE FUNCTION calculate_bonus(salary NUMERIC, bonus_percentage NUMERIC)
RETURNS NUMERIC
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN salary * (bonus_percentage / 100);
END;
$$;
```

### Calling the Function 
```sql
SELECT calculate_bonus(5000, 10);
```
**Output**  
```
500
```

## 4. Functions Returning Tables

Functions in PostgreSQL can **return entire tables** instead of just a single value.  

### Example 3: Returning a Table of High Salary Employees
```sql
CREATE FUNCTION get_high_salary_employees(threshold NUMERIC)
RETURNS TABLE(emp_id INT, first_name TEXT, last_name TEXT, salary NUMERIC)
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN QUERY 
    SELECT emp_id, first_name::text, last_name::text, salary
    FROM employees
    WHERE salary > threshold;
END;
$$;
```

### Calling the Function
```sql
SELECT * FROM get_high_salary_employees(50000);
```
**Output**  
```
emp_id | first_name | last_name | salary  
-------+------------+-----------+--------  
1      | Nakul       | Mitra       | 70000  
2      | Prince       | Chauhan     | 60000  
```

## 5. Managing Functions

### Modifying a Function
To modify a function, we must **drop the existing one** and recreate it.  
```sql
DROP FUNCTION calculate_annual_salary(NUMERIC);

CREATE FUNCTION calculate_annual_salary(monthly_salary NUMERIC)
RETURNS NUMERIC
LANGUAGE plpgsql
AS $$
BEGIN
    RETURN monthly_salary * 12;
END;
$$;
```

Second option is to use **CREATE AND REPLACE**
```sql
CREATE OR REPLACE FUNCTION calculate_annual_salary(monthly_salary NUMERIC)
RETURNS NUMERIC
LANGUAGE plpgsql
AS $$
BEGIN
    -- Updated logic
END;
$$;
```

### Dropping a Function
To delete a function permanently:  
```sql
DROP FUNCTION calculate_annual_salary(NUMERIC);
```

## 6. Use Cases of Functions in PostgreSQL
* `Data Validation` Functions can validate inputs before inserting them into a database.
* `Custom Calculations` E.g., Calculating tax, bonuses, or commissions in payroll systems.
* `Data Transformation`Transforming raw data into meaningful results, such as aggregations.
* `Reusable Business Logic` Encapsulating business rules inside functions ensures consistency.

## 7. Summary  
* Functions in PostgreSQL allow us to define reusable logic that returns values.  
* They differ from stored procedures as they always return values and cannot manage transactions.  
* Functions can return single values, multiple values, or even entire tables.
* They are useful for custom calculations, data validation, and reusable business logic.