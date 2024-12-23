# Creating and Managing Tables in PostgreSQL
Tables are the fundamental building blocks of any relational database, including PostgreSQL. They allow you to store, organize, and retrieve structured data efficiently. This guide explains the basics of creating, modifying, and managing tables in PostgreSQL, along with the use of constraints to enforce data integrity.

[![](https://markdown-videos-api.jorgenkh.no/youtube/vDAO_qISxT4)](https://youtu.be/vDAO_qISxT4)

## What is a Table in PostgreSQL?
A table in PostgreSQL is a structured collection of rows and columns used to store data. Each column in a table has a specific **data type**, which determines the kind of data it can hold, such as integers, text, or dates. Tables can also have constraints applied to their columns, ensuring the integrity and validity of the data.

### Example: Basic Table Structure
A simple `employees` table might look like this:
| emp_id | first_name | last_name | email | hire_date | salary |
| -------|------------|-----------|-------|-----------|--------|
|    1   |  Aman  |  Tiwari  | aman@gmail.com | 2024-01-15 | 55000.00 |
|    2   |  Harshada  |  Kute  | hk@gmail.com | 2023-04-10 | 45000.00 |

## 1. Creating Tables
To create a table in PostgreSQL, use the `CREATE TABLE` command. Each column is defined with a name, data type, and optional constraints.

### Basic Syntax
```
CREATE TABLE table_name (
    column_name data_type constraints
);
```

### Example: Creating the employees Table
```
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50),
    email VARCHAR(100) UNIQUE NOT NULL,
    hire_date DATE DEFAULT CURRENT_DATE,
    salary NUMERIC(10, 2) CHECK (salary > 0)
);
```

### Explanation of Columns:
#### 1. emp_id SERIAL PRIMARY KEY:
* `SERIAL:` Auto-generates a unique integer value for each row.
* `PRIMARY KEY:` Ensures that `emp_id` is unique and not null.

#### 2. first_name VARCHAR(50) NOT NULL:
* `VARCHAR(50):` Stores up to 50 characters.
* `NOT NULL:` Ensures the column cannot have null values.

#### 3. email VARCHAR(100) UNIQUE NOT NULL:
* `UNIQUE:` Ensures no two rows have the same email value.
* `NOT NULL:` Guarantees an email value is provided for every employee.

#### 4. hire_date DATE DEFAULT CURRENT_DATE:
* `DEFAULT CURRENT_DATE:` Automatically sets the hire date to the current date if no value is provided.

#### 5. salary NUMERIC(10, 2) CHECK (salary > 0):
* `NUMERIC(10, 2):` Stores up to 10 digits, with 2 digits after the decimal point.
* `CHECK (salary > 0):` Validates that the salary is always a positive value.

## 2. Modifying Tables
PostgreSQL allows you to modify an existing table using the `ALTER TABLE` command.

### Add a Column
```
ALTER TABLE employees ADD COLUMN department VARCHAR(50);
```
Adds a new column named `department` to the `employees` table.

### Modify a Column
```
ALTER TABLE employees ALTER COLUMN salary SET NOT NULL;
```
Ensures the `salary` column cannot have null values.

### Remove a Column
```
ALTER TABLE employees DROP COLUMN department;
```
Deletes the `department` column from the `employees` table.

## 3. Using Constraints
Constraints are rules applied to columns to ensure data accuracy and integrity. PostgreSQL supports several types of constraints:

### Common Constraints
#### 1. Primary Key:
* Ensures each row has a unique identifier.
* Example:
```
emp_id SERIAL PRIMARY KEY
```

#### 2. Foreign Key:
* Establishes a relationship between two tables.
* Example:
```
FOREIGN KEY (manager_id) REFERENCES employees(emp_id)
```

#### 3. Unique:
* Ensures no duplicate values in a column.
* Example:
```
email VARCHAR(100) UNIQUE
```

#### 4. Not Null:
* Ensures a column cannot have null values.
* Example:
```
first_name VARCHAR(50) NOT NULL
```

#### 5. Check:
* Validates a specific condition.
* Example:
```
salary NUMERIC(10, 2) CHECK (salary > 0)
```

### Example: Creating a departments Table
```
CREATE TABLE departments (
    dept_id SERIAL PRIMARY KEY,
    dept_name VARCHAR(100) NOT NULL UNIQUE,
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES employees(emp_id)
);
```

#### Explanation:
* `dept_id SERIAL PRIMARY KEY:` Unique identifier for each department.
* `dept_name VARCHAR(100) NOT NULL UNIQUE:` Ensures each department has a unique name.
* `FOREIGN KEY (manager_id) REFERENCES employees(emp_id):` Links manager_id in the departments table to emp_id in the employees table.

## 4. Practical Example: CRUD Operations
Let’s see how to perform basic CRUD (Create, Read, Update, Delete) operations using the tables we created.

### Insert Data
```
INSERT INTO employees (first_name, last_name, email, hire_date, salary) 
VALUES 
('Aman', 'Tiwari', 'aman@gmail.com', '2024-01-15', 55000.00),
('Harshada', 'Kute', 'hk@gmail.com', DEFAULT, 45000.00);
```

```
INSERT INTO departments (dept_name, manager_id) 
VALUES 
('HR', 1),
('Finance', 2);
```

### Retrieve Data
```
SELECT * FROM employees;
```

### Update Data
```
UPDATE employees
SET salary = 60000
WHERE emp_id = 1;
```

### Delete Data
```
DELETE FROM employees
WHERE emp_id = 2;
```

## 5. Summary
* **Tables** are essential for organizing and storing data in PostgreSQL.
* Use constraints like `PRIMARY KEY`, `FOREIGN KEY`, and `UNIQUE` to enforce data integrity.
* The `ALTER TABLE` command allows easy modification of table structures.
* Perform CRUD operations to manage data in your tables.