# SQL Basics with PostgreSQL
This document explores the fundamental CRUD operations in SQL, specifically tailored for PostgreSQL. CRUD stands for **Create**, **Read**, **Update**, and **Delete**, which are the basic operations to manage data in a database. These operations are the building blocks of any database system, allowing users to store, retrieve, modify, and delete data efficiently.

[![](https://markdown-videos-api.jorgenkh.no/youtube/4Coyz00SINg)](https://youtu.be/4Coyz00SINg)

## 1. What is CRUD?
CRUD is an acronym for the four primary operations performed on a database table:
* `Create:` Add new records to a table using the `INSERT` statement.
* `Read:` Retrieve data from a table using the `SELECT` statement.
* `Update:` Modify existing records in a table using the `UPDATE` statement.
* `Delete:` Remove records from a table using the `DELETE` statement.

These operations allow databases to store, manage, and organize data effectively, providing the foundation for any database-driven application.

## 2. The SELECT Statement (Read Operation)
The `SELECT` statement is used to query and retrieve data from a table. It is the most commonly used SQL operation for viewing and analyzing data.
### Basic Syntax
```sql
SELECT column1, column2, ...  
FROM table_name  
WHERE condition;  
```

### Examples
* Retrieve all columns from a table
```sql
SELECT * FROM employees;  
```

* Retrieve specific columns from a table
```sql
SELECT first_name, salary FROM employees;  
```

* Retrieve data with a condition
```sql
SELECT first_name, salary FROM employees  
WHERE salary > 50000;  
```

* Sort data using ORDER BY
```sql
SELECT first_name, hire_date FROM employees  
ORDER BY hire_date DESC;  
```

### Key Notes
* Use the `WHERE` clause to filter records based on conditions.
* Use `ORDER` BY to sort the data in ascending (`ASC`) or descending (`DESC`) order.

## 3. The INSERT Statement (Create Operation)
The `INSERT` statement is used to add new rows to a table. It allows you to populate tables with data.
### Basic Syntax
```sql
INSERT INTO table_name (column1, column2, ...)  
VALUES (value1, value2, ...);  
```

### Examples
* Insert a new row
```sql
INSERT INTO employees (first_name, last_name, email, hire_date, salary)  
VALUES ('Nakul', 'Mitra', 'nakul@gmail.com', '2021-03-01', 65000);  
```

* Insert with default values
```sql
INSERT INTO employees (first_name, email, salary)  
VALUES ('Aman', 'aman@gmail.com', 48000);  
```

### Key Notes
* Always specify the columns when inserting data to ensure clarity and prevent errors.
* Omitted columns must either allow `NULL` values or have default values defined.

## 4. The UPDATE Statement (Update Operation)
The `UPDATE` statement modifies existing records in a table. This operation is particularly useful for correcting or altering data.
### Basic Syntax
```sql
UPDATE table_name  
SET column1 = value1, column2 = value2, ...  
WHERE condition;  
```

### Examples
* Update a specific record
```sql
UPDATE employees  
SET salary = 70000  
WHERE emp_id = 1;  
```

* Update multiple records
```sql
UPDATE employees  
SET salary = salary * 1.10  
WHERE salary < 50000;  
```

* Update all records (use with caution)
```sql
UPDATE employees  
SET salary = 60000;  
```

### Key Notes
* Always include a `WHERE` clause to avoid updating all rows unintentionally.
* Verify conditions carefully to ensure accuracy.

## 5. The DELETE Statement (Delete Operation)
The `DELETE` statement removes rows from a table. It is essential to use the `WHERE` clause to avoid accidentally deleting all data.
### Basic Syntax
```sql
DELETE FROM table_name  
WHERE condition;  
```

### Examples
* Delete a specific record
```sql
DELETE FROM employees  
WHERE emp_id = 2;  
```

* Delete multiple records
```sql
DELETE FROM employees  
WHERE salary < 40000;  
```

* Delete all rows (use with caution)
```sql
DELETE FROM employees;  
```

### Key Notes
* Deleting all rows does not delete the table structure. The table remains empty after the operation.
* Use the `WHERE` clause to specify the rows to be deleted.

## 6. Practical Example: Full CRUD Cycle
**Scenario: Managing Departments**
### 1. Create a New Department
```sql
INSERT INTO departments (dept_name)  
VALUES ('Marketing');  
```

### 2. Read All Departments
```sql
SELECT * FROM departments;  
```

### 3. Update Department Name
```sql
UPDATE departments  
SET dept_name = 'Sales and Marketing'  
WHERE dept_name = 'Marketing';  
```

### 4. Delete the Department
```sql
DELETE FROM departments  
WHERE dept_name = 'Sales and Marketing';  
```

## 7. Key Takeaways
### 1. CRUD Operations:
* `SELECT:` Retrieve data.
* `INSERT:` Add new data.
* `UPDATE:` Modify existing data.
* `DELETE:` Remove unwanted data.

### 2.Best Practices:
* Always use the `WHERE` clause with `UPDATE` and `DELETE` statements to avoid unintended changes.
* Validate data before inserting it to maintain data integrity.
* Use descriptive column names to enhance query readability and maintainability.

## 8. Summary
CRUD operations are the foundation of database management, allowing you to effectively interact with and maintain your data.