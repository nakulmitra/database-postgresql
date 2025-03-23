## Overview
Performance optimization is crucial in PostgreSQL to ensure efficient query execution and minimal resource consumption. This guide explores:

* What are dead and live tuples?
* How dead tuples impact database performance
* Using `VACUUM`, `ANALYZE`, and `AUTOVACUUM` to clean up dead tuples
* Reindexing to improve query performance
* Best practices for PostgreSQL performance optimization

## Understanding Dead and Live Tuples

### What are Tuples?
In PostgreSQL, **tuples** represent rows in a table. Whenever data is **INSERTED, UPDATED, or DELETED**, PostgreSQL manages tuples using a technique called **MVCC (Multi-Version Concurrency Control)**.

### What are Live Tuples?
- **Live tuples** are active rows that are currently valid and visible to queries.
- They represent the **latest committed version** of a row in the database.

### What are Dead Tuples?
- **Dead tuples** are outdated row versions that are **no longer needed** but still exist in the table.
- These occur when:
  - A row is updated → The **old version** remains as a dead tuple.
  - A row is deleted → The row is marked as deleted but **not immediately removed**.

### Example of Dead Tuples Creation

```
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    salary NUMERIC(10,2)
);

-- Insert a row
INSERT INTO employees (name, salary) VALUES ('Nakul', 50000);

-- Update the salary (this creates a dead tuple)
UPDATE employees SET salary = 60000 WHERE name = 'Nakul';

-- Delete a row (leaves a dead tuple)
DELETE FROM employees WHERE name = 'Nakul';
```

### Why Do Dead Tuples Exist?
PostgreSQL follows **MVCC (Multi-Version Concurrency Control)**, meaning:
- Each transaction works with a **snapshot** of the data.
- When a row is updated, PostgreSQL **does not modify the row directly**. Instead, it **creates a new version** and marks the old one as **dead**.
- Deleted rows **are not immediately removed** but are simply **marked as dead tuples**.

## How Dead Tuples Impact Database Performance
Dead tuples **consume disk space** and slow down queries because:
* PostgreSQL must scan through **dead tuples** to fetch valid data.
* **Indexes grow larger**, making queries slower.
* **Tables and indexes become bloated**, increasing storage requirements.

### Checking Dead Tuples in a Table
We can check for dead tuples using the `pg_stat_user_tables` view:

```
SELECT relname AS table_name, n_live_tup, n_dead_tup 
FROM pg_stat_user_tables 
WHERE schemaname = 'public';
```

**Columns explained:**
* `n_live_tup` → Number of **live tuples** (valid rows).
* `n_dead_tup` → Number of **dead tuples** (wasted space).
* If `n_dead_tup` is high, cleaning up the table is necessary.

## Using VACUUM to Clean Up Dead Tuples
### The VACUUM Command
The `VACUUM` command removes dead tuples and **reclaims disk space**.

* **Basic Vacuum Command**
```
VACUUM employees;
```
* Removes dead tuples and **marks space as reusable**.
* **Does NOT** update statistics for query planner.

### VACUUM ANALYZE - Cleaning + Updating Statistics
`VACUUM ANALYZE` removes dead tuples **and updates statistics** for better query planning.

* **Run Vacuum Analyze**
```
VACUUM ANALYZE employees;
```
* **Improves query performance** by updating PostgreSQL's query planner.

### AUTOVACUUM - Automatic Dead Tuple Cleanup
PostgreSQL has an **autovacuum** process that runs automatically to clean dead tuples.

* **Checking Autovacuum Activity**
```
SELECT relname, last_autovacuum
FROM pg_stat_user_tables 
WHERE last_autovacuum IS NOT NULL;
```

* If `last_autovacuum` is NULL, autovacuum **is not running** frequently.

* **Configuring Autovacuum for Better Performance**
If autovacuum is not cleaning up dead tuples efficiently, adjust **postgresql.conf** settings:

```
autovacuum = on
autovacuum_vacuum_threshold = 50
autovacuum_vacuum_cost_limit = 2000
```

We can also **manually configure autovacuum** for a specific table:

```
ALTER TABLE employees
SET (autovacuum_vacuum_threshold = 1000, autovacuum_vacuum_scale_factor = 0.0);
```

## Re-Indexing for Performance Optimization

### Why Do Indexes Need Reindexing?
Indexes grow **over time** due to **updates and deletes**.
- PostgreSQL **does NOT automatically shrink indexes**, leading to **index bloat**.
- **A bloated index makes queries slower.**

### Checking for Index Bloat
We can check for bloated indexes using:

```
SELECT 
    schemaname, 
    relname AS table_name, 
    indexrelname AS index_name, 
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan > 0;
```

* If **index sizes** are **too large**, re-indexing is required!

### Running REINDEX to Optimize Performance

* **Reindexing a Single Index**
```
REINDEX INDEX employees_name_idx;
```

* **Reindexing an Entire Table**
```
REINDEX TABLE employees;
```

* **Reindexing the Entire Database**
```
REINDEX DATABASE mydatabase;
```

* **When to Run REINDEX?**
* If query performance **slows down** due to large index sizes.
* If `pg_stat_user_indexes` shows **large indexes**.
* If `VACUUM` does not **significantly improve performance**.

## Best Practices for PostgreSQL Performance Optimization

* **Enable Autovacuum**
Ensure autovacuum is enabled to **automatically clean dead tuples**.

* **Schedule Manual VACUUM for Large Tables**
For large tables, schedule a manual `VACUUM` **during off-peak hours**:

```
VACUUM ANALYZE my_large_table;
```

* **Monitor Dead Tuples Regularly**
Check dead tuples periodically:

```
SELECT relname, n_live_tup, n_dead_tup 
FROM pg_stat_user_tables;
```

* **Use Reindexing for Large Indexes**
If indexes become **too large**, use:

```
REINDEX TABLE my_table;
```

## Summary
* **Dead tuples** slow down queries and increase storage usage.
* **VACUUM and ANALYZE** clean up dead tuples and update query planner statistics.
* **AUTOVACUUM** helps PostgreSQL automatically manage dead tuples.
* **Reindexing** keeps indexes **efficient** and maintains **query performance**.