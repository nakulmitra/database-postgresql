# Understanding Stored Procedures in PostgreSQL
A **Stored Procedure** in PostgreSQL is a reusable set of SQL statements that are stored in the database and executed as a unit. Unlike functions, stored procedures **do not return values** but can execute **multiple SQL statements** together, including **transaction control statements** like `COMMIT` and `ROLLBACK`.  

[![](https://markdown-videos-api.jorgenkh.no/youtube/OqrwTHCFItA)](https://youtu.be/OqrwTHCFItA)

## 1. Why Use Stored Procedures?
Stored procedures are useful because they:  
* Automate complex database operations  
* Reduce network overhead by executing logic directly in the database  
* Improve performance by reducing multiple round-trips between the application and database  
* Help in transaction management (ensuring data integrity)  
* Enforce business logic consistently across an application

## 2. Difference Between Functions and Stored Procedures in PostgreSQL

| Feature              | Stored Procedure | Function |
|----------------------|----------------|----------|
| **Return Value**     | Does not return any value | Returns a value (scalar, table, or composite type) |
| **Transaction Control** | Can use `BEGIN`, `COMMIT`, `ROLLBACK` | Cannot manage transactions |
| **Use Case**        | Best for data manipulation (`INSERT`, `UPDATE`, `DELETE`) | Best for computations and returning results |

## 3. Creating a Simple Stored Procedure in PostgreSQL
### Example: A Procedure to Increase Employee Salaries
```
CREATE PROCEDURE update_salaries(IN percent_increase NUMERIC)
LANGUAGE plpgsql
AS $$
BEGIN
    UPDATE employees
    SET salary = salary + (salary * percent_increase / 100);
    RAISE NOTICE 'Salaries updated successfully by % percent.', percent_increase;
END;
$$;
```

### Explanation:
- `CREATE PROCEDURE update_salaries(IN percent_increase NUMERIC)`: Defines the procedure with an input parameter.  
- `UPDATE employees SET salary = salary + (salary * percent_increase / 100)`: Increases salaries based on the given percentage.  
- `RAISE NOTICE`: Prints a success message in the console.

## 4. Calling a Stored Procedure
Unlike functions, procedures are executed using the `CALL` statement:
```
CALL update_salaries(10);
```

This increases employee salaries by **10%**.

## 5. Transaction Control in Stored Procedures
Stored procedures in PostgreSQL **can manage transactions** using `COMMIT` and `ROLLBACK`, which functions cannot.  

### Example: Using Transaction Control
```
CREATE PROCEDURE update_salaries_with_transaction(IN percent_increase NUMERIC)
LANGUAGE plpgsql
AS $$
BEGIN
    BEGIN -- Explicitly starting a new transaction block
        UPDATE employees
        SET salary = salary + (salary * percent_increase / 100);

        -- Condition check if the percentage increase is too high
        IF percent_increase > 20 THEN
            ROLLBACK;  -- Undo the update operation
            RAISE NOTICE 'Transaction rolled back: Too high a percentage!';
        ELSE
            COMMIT;  -- Save the update operation
            RAISE NOTICE 'Transaction committed successfully.';
        END IF;
    END;
END;
$$;
```

### How It Works? 
1. `BEGIN` starts a new transaction.  
2. The `UPDATE` query increases salaries.  
3. If `percent_increase > 20`, `ROLLBACK` is triggered to undo the changes.  
4. Otherwise, `COMMIT` saves the changes permanently.

## 6. Modifying and Dropping Stored Procedures

### 6.1 Modifying a Procedure
Stored procedures can be modified by **dropped and recreated**.
```
DROP PROCEDURE update_salaries;

CREATE PROCEDURE update_salaries(IN percent_increase NUMERIC)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Updated logic here
END;
$$;
```

Use **CREATE OR REPLACE**
```
CREATE OR REPLACE PROCEDURE update_salaries(IN percent_increase NUMERIC)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Updated logic here
END;
$$;
```

### 6.2 Dropping a Procedure
```
DROP PROCEDURE update_salaries;
```

This removes the procedure from the database.

## 7. Use Cases of Stored Procedures
1. **Batch Operations**  
- Updating salaries for all employees  
- Deleting old records automatically  

2. **Transaction Handling**  
- Ensuring data integrity when performing multiple dependent operations  

3. **Data Processing and Automation**  
- Generating reports by aggregating data  
- Automating periodic database updates

## 8. Summary
* Stored Procedures automate repetitive database tasks.  
* Unlike functions, procedures can **manage transactions** (`COMMIT`, `ROLLBACK`).  
* Use the `CALL` statement to execute a procedure.  
* Procedures must be dropped and recreated for modifications.  
* Best used for batch updates, transaction control, and enforcing business logic.