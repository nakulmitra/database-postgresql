# PostgreSQL Tutorial - Why PostgreSQL Uses Sequential Scan Even If Index Exists

PostgreSQL is known for its intelligent and cost-based query planner. However, many developers are surprised when PostgreSQL performs a **Sequential Scan (Seq Scan)** even though an index exists on the filtered column. Understanding why this happens is essential for effective **query performance tuning** and **indexing strategy**.

## 1. What is a Sequential Scan?

A **Sequential Scan** (also called `Seq Scan`) is the most basic method PostgreSQL uses to read data. It reads the entire table row by row, even if there is a WHERE clause.

### It usually occurs when:

* No suitable index exists.
* PostgreSQL estimates that a full table scan is cheaper than using the index.

## 2. When Does PostgreSQL Choose Index Scan?

PostgreSQL performs an **Index Scan** when the **cost of using the index is lower** than doing a full scan. This typically happens when:

* The query has a **WHERE condition** on a column that is indexed.
* The condition is **highly selective** (matches few rows).
* PostgreSQL **estimates fewer disk reads** using the index.

**Example:**

```sql
SELECT * FROM tutorial.employees WHERE id = 4;
```

If `id` is a primary key or has an index, PostgreSQL usually uses an Index Scan here.

## 3. Why PostgreSQL Might Ignore the Index

Even when an index exists, PostgreSQL may still prefer a Sequential Scan for the following reasons:

### a. Low Selectivity

If the WHERE clause matches **a large number of rows**, PostgreSQL avoids the overhead of jumping between the index and the table heap.

```sql
SELECT * FROM employees WHERE dept_id = 3;
```

If `dept_id = 3` matches millions of rows, a sequential scan might be faster.

### b. Small Table Size

For small tables, the entire table can fit in memory or be scanned quickly. In such cases, the planner prefers a Seq Scan as it’s simpler and faster.

### c. Stale or Missing Statistics

PostgreSQL relies on **table statistics** to make planning decisions. If stats are outdated, it may **misestimate** the number of matching rows.

Fix:

```sql
ANALYZE employees;
```

Also, running `VACUUM ANALYZE` periodically helps maintain up-to-date stats and the visibility map (useful for index-only scans).

### d. Parallel Execution

PostgreSQL can **parallelize sequential scans** but not regular index scans. For very large tables, a **Parallel Seq Scan** is faster than using an index.

### e. Function Usage or Type Mismatch

If a function is applied to the indexed column, PostgreSQL **cannot use the index** unless it is a **functional index**.

**Example - index not used:**

```sql
SELECT * FROM employees WHERE UPPER(full_name) = 'EMPLOYEE_125';
```

**Fix - use a functional index:**

```sql
CREATE INDEX idx_upper_name ON employees (UPPER(full_name));
```

## 4. Real-World Example with EXPLAIN ANALYZE

Let's take an example with 50 million rows:

```sql
EXPLAIN ANALYZE
SELECT * FROM tutorial.employees WHERE dept_id = 3;
```

**Query Plan Output:**

```
->  Parallel Seq Scan on employees  (cost=0.00..667407.90 rows=4189562 width=0)
    Filter: (dept_id = 3)
    Rows Removed by Filter: 13332151
```

### Key Points:

* **Parallel Seq Scan** was chosen.
* **3 workers** processed data in parallel.
* The planner estimated it’s cheaper than an Index Scan.

## 5. How to Force Index Usage (Only for Testing)

You can force PostgreSQL to use an index by disabling sequential scans:

```sql
SET enable_seqscan = OFF;
```

* **Warning:** Do this only for debugging or benchmarking. Never use this in production as it bypasses the cost-based optimization.

## 6. Summary

| Scenario                          | PostgreSQL Behavior  | Recommendation                       |
| --------------------------------- | -------------------- | ------------------------------------ |
| Query matches many rows           | Uses Sequential Scan | Consider if index is beneficial      |
| Table is small                    | Uses Sequential Scan | Acceptable behavior                  |
| Statistics are stale              | May misestimate cost | Run `ANALYZE` regularly              |
| Function used in WHERE clause     | Index not used       | Create a functional index            |
| Planner thinks Seq Scan is faster | Ignores index        | Use `EXPLAIN ANALYZE` to verify plan |

## Note on Query Planner

PostgreSQL uses a **cost-based query planner** to decide the best execution strategy. It calculates:

* **Startup Cost**: Cost before first row is returned.
* **Total Cost**: Cost to return all rows.
* **Estimated Rows**: Number of rows expected to be returned.

The planner will **always choose the plan with the lowest total cost**, which sometimes means **ignoring indexes**.