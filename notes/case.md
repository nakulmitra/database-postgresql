# Introduction to CASE Statements in SQL  
The CASE statement in SQL is a powerful tool that allows you to add conditional logic directly into your queries. It works like an if-else or switch-case structure in programming languages, enabling you to perform conditional evaluations and return specific results based on defined conditions. This functionality is crucial for dynamic query customization, data categorization, and conditional formatting.

[![](https://markdown-videos-api.jorgenkh.no/youtube/lMEn2ZkWNNY)](https://youtu.be/5M8SoHw10Fk)

## 1. Syntax of CASE Statement  
The basic syntax of the CASE statement is:  

```sql
CASE  
    WHEN condition1 THEN result1  
    WHEN condition2 THEN result2  
    ...  
    [ELSE default_result]  
END  
```

### Explanation of Syntax  
- WHEN condition THEN result: Evaluates a condition. If the condition is true, the corresponding result is returned.  
- ELSE default_result: Optional. Specifies a default value if no conditions are true.  
- END: Marks the end of the CASE statement.  

> Note: If no conditions are met and there's no ELSE clause, the CASE statement will return `NULL`.  

## 2. Types of CASE Statements  
### Simple CASE  
Compares an expression to a set of predefined values.  
```sql
CASE expression  
    WHEN value1 THEN result1  
    WHEN value2 THEN result2  
    ELSE default_result  
END  
```  

### Searched CASE  
Evaluates multiple conditions independently.  
```sql
CASE  
    WHEN condition1 THEN result1  
    WHEN condition2 THEN result2  
    ELSE default_result  
END  
```

## 3. Practical Use Cases of CASE Statements  
### Example Table: Employees  
| emp_id | name       | salary   | department_name |  
|--------|------------|----------|-----------------|  
| 1      | Aman       | 25000    | HR              |  
| 2      | Janet      | 40000    | Finance         |  
| 3      | Harshada   | 55000    | IT              |  
| 4      | Shreya     | 70000    | HR              |  

### 3.1 CASE in SELECT Statement  
Categorize employees based on their salary levels:  

```sql
SELECT 
    emp_id,  
    name,  
    salary,  
    CASE  
        WHEN salary < 30000 THEN 'Low'  
        WHEN salary BETWEEN 30000 AND 50000 THEN 'Medium'  
        ELSE 'High'  
    END AS salary_category  
FROM employees;  
```  

Output:  
| emp_id | name       | salary | salary_category |  
|--------|------------|--------|-----------------|  
| 1      | Aman       | 25000  | Low             |  
| 2      | Janet      | 40000  | Medium          |  
| 3      | Harshada   | 55000  | High            |  
| 4      | Shreya     | 70000  | High            |  

Explanation:  
- Each employee is categorized into `Low`, `Medium`, or `High` salary brackets based on the CASE logic.  

### 3.2 CASE in Aggregate Functions  
Calculate total salary by salary categories:  

```sql
SELECT 
    CASE  
        WHEN salary < 30000 THEN 'Low'  
        WHEN salary BETWEEN 30000 AND 50000 THEN 'Medium'  
        ELSE 'High'  
    END AS salary_category,  
    SUM(salary) AS total_salary  
FROM employees  
GROUP BY  
    CASE  
        WHEN salary < 30000 THEN 'Low'  
        WHEN salary BETWEEN 30000 AND 50000 THEN 'Medium'  
        ELSE 'High'  
    END;  
```  

Output:  
| salary_category | total_salary |  
|-----------------|--------------|
| Low             | 25000        |  
| Medium          | 40000        |  
| High            | 125000       |  

### 3.3 CASE in WHERE Clause  
Filter employees based on dynamic conditions:  

```sql
SELECT emp_id, name, salary  
FROM employees  
WHERE  
    CASE  
        WHEN salary < 30000 THEN 'Low'  
        ELSE 'Other'  
    END = 'Low';  
```  

Explanation: This query retrieves only employees with salaries categorized as `Low`.

### 3.4 CASE in ORDER BY Clause  
Sort employees by salary category priority:  

```sql
SELECT emp_id, name, salary  
FROM employees  
ORDER BY  
    CASE  
        WHEN salary < 30000 THEN 1  
        WHEN salary BETWEEN 30000 AND 50000 THEN 2  
        ELSE 3  
    END;  
```  

Explanation:  
- Employees are sorted into priority levels (1 for Low, 2 for Medium, and 3 for High salary categories).  

### 3.5 Formatting Output with CASE  
Add conditional formatting to display salaries with a currency symbol:  

```sql
SELECT  
    emp_id,  
    name,  
    CASE  
        WHEN salary < 50000 THEN '$' || salary  
        ELSE '$' || salary * 1.10  
    END AS formatted_salary  
FROM employees;  
```  

Output:  
| emp_id | name       | formatted_salary |  
|--------|------------|------------------|
| 1      | Aman       | $25000           |  
| 2      | Janet      | $40000           |  
| 3      | Harshada   | $60500           |  
| 4      | Shreya     | $77000           |  

## 4. Advantages of CASE Statements  
- Dynamic Querying: Enables conditional logic within SQL queries.  
- Data Categorization: Simplifies grouping and filtering of data.  
- Custom Formatting: Allows formatting of output values based on conditions.  
- Versatility: Can be used in SELECT, WHERE, ORDER BY, GROUP BY, and HAVING clauses.  



## 5. Best Practices for Using CASE Statements  
1. Minimize Complexity: Avoid overly nested CASE statements for better readability.  
2. Test for NULL: Always handle NULL values to avoid unexpected results.  
3. Use Consistent Conditions: Ensure consistent logic when combining CASE with GROUP BY or ORDER BY.  
4. Optimize Performance: For large datasets, consider using indexed columns in CASE conditions.  



## 6. Summary  
- CASE statements bring the power of conditional logic to SQL.  
- They can be used in various parts of a query, including SELECT, WHERE, ORDER BY, and GROUP BY.  
- Practical use cases include data categorization, dynamic filtering, custom formatting, and conditional aggregation.  

By mastering CASE statements, you can write more flexible, readable, and efficient SQL queries, making them an essential tool for any database developer or data analyst.  