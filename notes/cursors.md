# Understanding Cursors in PostgreSQL
In PostgreSQL, a **cursor** is a database object used to retrieve query results one row at a time rather than fetching the entire result set at once. Cursors are especially useful when working with large datasets, as they optimize memory usage and allow row-by-row processing within **PL/pgSQL** functions and procedures.

## 1. What Are Cursors in PostgreSQL?
### Definition
A cursor in PostgreSQL is a pointer that allows traversal over a result set **one row at a time**. This is particularly useful in **procedural operations**, such as **loops**, where we need to process data row by row.

### Why Use Cursors?
* Allow **iterative row processing**, reducing memory overhead.
* Are useful for handling **large datasets** efficiently.
* Provide control over **fetching data** (one row at a time, in batches, etc.).
* Can be used inside **PL/pgSQL** functions for procedural processing.

## 2. Cursor Lifecycle in PostgreSQL
A cursor follows a **four-step lifecycle** in PostgreSQL:

| Step | Description |
|------|-------------|
| **1. Declare** | Define a cursor and associate it with a query. |
| **2. Open** | Execute the query and make the cursor ready to fetch results. |
| **3. Fetch** | Retrieve rows from the cursor (one at a time or in batches). |
| **4. Close** | Release resources by closing the cursor. |

## 3. Declaring and Using Cursors in PostgreSQL
### Basic Cursor Syntax
A cursor is declared using the `DECLARE` statement inside a **PL/pgSQL block**:
```
DECLARE cursor_name CURSOR FOR query;
```
After declaring, it must be **opened**, **fetched**, and **closed**.

#### PL/pgSQL Function to Iterate Over Rows Using a Cursor
```
CREATE OR REPLACE FUNCTION iterate_employees()
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    emp_cursor CURSOR FOR SELECT emp_id, first_name, last_name, salary FROM employees;
    emp_record RECORD;
BEGIN
    OPEN emp_cursor; -- Open the cursor

    LOOP
        FETCH emp_cursor INTO emp_record; -- Fetch a row into emp_record
        EXIT WHEN NOT FOUND; -- Exit when all rows are fetched

        -- Process each row
        RAISE NOTICE 'Employee Id: %, Name: % %, Salary: %', 
                     emp_record.emp_id, emp_record.first_name, emp_record.last_name, emp_record.salary;
    END LOOP;

    CLOSE emp_cursor; -- Close the cursor
    RETURN 'true';
END;
$$;
```

### Explanation
* `DECLARE:` Defines the cursor with a query.
* `OPEN:` Executes the query and prepares the cursor for fetching data.
* `FETCH:` Retrieves one row at a time into `emp_record`.
* `EXIT WHEN NOT FOUND:` Stops the loop when all rows are fetched.
* `CLOSE:` Releases resources associated with the cursor.

**Calling the Function:**
```
SELECT * from iterate_employees();
```

**Sample Output:**
```
NOTICE: Employee Id: 1, Name: Nakul Mitra, Salary: 60000
NOTICE: Employee Id: 2, Name: Piyush Mudgal, Salary: 50000
...

```

## 4. Using Cursors with Conditional Logic
Cursors can be used to process data conditionally. Let's say we want to **increase the salary** of employees earning above a certain threshold.

### Example: Conditional Updates Using a Cursor
```
CREATE OR REPLACE FUNCTION update_high_salary_employees(threshold NUMERIC, bonus NUMERIC)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    emp_cursor CURSOR FOR SELECT emp_id, salary FROM employees WHERE salary > threshold;
    emp_row RECORD;
BEGIN
    OPEN emp_cursor;

    LOOP
        FETCH emp_cursor INTO emp_row;
        EXIT WHEN NOT FOUND;

        -- Update salary
        UPDATE employees
        SET salary = emp_row.salary + bonus
        WHERE emp_id = emp_row.emp_id;

        RAISE NOTICE 'Updated Employee Id: %, New Salary: %', emp_row.emp_id, emp_row.salary + bonus;
    END LOOP;

    CLOSE emp_cursor;
    RETURN 'true';
END;
$$;
```

**Calling the Function:**
```
SELECT * from update_high_salary_employees(45000, 1000);
```

**Sample Output:**
```
NOTICE: Updated Employee Id: 1, New Salary: 61000
NOTICE: Updated Employee Id: 2, New Salary: 51000
...

```

## 5. Managing Cursors Explicitly
We can also define and manage **cursors explicitly** within an anonymous PL/pgSQL block as well.  

### Example: Explicit Cursor Management
```
DO $$
DECLARE
    emp_cursor CURSOR FOR SELECT emp_id, first_name FROM employees;
    emp_row RECORD;
BEGIN
    OPEN emp_cursor;

    LOOP
        FETCH emp_cursor INTO emp_row;
        EXIT WHEN NOT FOUND;

        RAISE NOTICE 'Employee ID: %, Name: %', emp_row.emp_id, emp_row.first_name;
    END LOOP;

    CLOSE emp_cursor;
END;
$$;
```

## 6. Best Practices for Using Cursors in PostgreSQL
* `Use Cursors Only When Necessary:` If a **set-based** operation can achieve the same result, use it instead (e.g., `UPDATE` instead of looping over rows).
* `Always Close Cursors:` Leaving cursors open can lead to resource leaks and performance issues.
* `Optimize Queries for Cursors:` Ensure that the query inside the cursor retrieves only the necessary data (use `WHERE`, `LIMIT`).
* `Avoid Fetching Large Datasets:` If you need to process large datasets, consider **batch processing** instead of fetching one row at a time.
* `Prefer FOR Loops for Simple Iteration:` PostgreSQL allows you to loop directly over queries without explicitly using cursors.

## 7. Summary
* **Cursors in PostgreSQL** allow row-by-row traversal of query results.
* They are useful for **iterative operations** and handling **large datasets efficiently**.
* The **cursor lifecycle** includes **DECLARE, OPEN, FETCH, and CLOSE**.