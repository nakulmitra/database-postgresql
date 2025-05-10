# Why PostgreSQL Uses Sequential Scan Even If Index Exists

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

## 4. Real-World Example - Parallel Sequential Scan Despite Index

In this example, we run the following query on a table with 5 crore (50 million) records:

```sql
EXPLAIN ANALYZE SELECT COUNT(1) FROM tutorial.employees WHERE dept_id = 4;
```

Even though the `dept_id` column is indexed, PostgreSQL chooses a **Parallel Sequential Scan** rather than using the index.

Let's break down why and what the execution plan reveals

### Screenshot 1: Query Execution Plan (Execution Time = 8653 ms)
![Sequential Scan Query Planner](https://github.com/nakulmitra/database-postgresql/blob/master/notes/images/Sequential_Scan_Query_Planner_1.png)

Key Highlights:

* **Parallel Seq Scan on employees**
  PostgreSQL reads the full `employees` table in parallel across 3 processes (1 main + 2 workers), scanning all rows to find those with `dept_id = 4`.

* **Rows Removed by Filter: 13,333,128**
  Out of \~50 million rows, over 13 million were evaluated and removed by the filter.

* **Rows Returned: 3,333,538 (approx.)**
  That's the number of rows that matched the condition `dept_id = 4`.

* **Execution Time: \~8653 ms**
  The query took over 8.5 seconds to execute.

### Screenshot 2: Same Query After Some Optimizations (Execution Time = 7156 ms)
![Sequential Scan Query Planner](https://github.com/nakulmitra/database-postgresql/blob/master/notes/images/Index_Scan_Query_Planner_1.png)

This screenshot is from the same query, after creating the index.

Key Changes:

* Execution Time: ~7156 ms
* Scan Type: Still a Parallel Sequential Scan
* Workers Used: 2
* Query Plan: Did not switch to index scan despite index being available.

### Why PostgreSQL Still Ignored the Index

Even though an index on `dept_id` was present in the second query:

1. **Large Matching Dataset (\~3.3 million rows)**
   PostgreSQL estimated that a large portion of the table matched the filter (`dept_id = 4`). In such cases, using an index becomes *less efficient* due to:

   * The overhead of reading many index entries.
   * The need to fetch corresponding heap (table) blocks, resulting in random I/O.

2. **Parallel Sequential Scan is Cheaper for Large Reads**
   PostgreSQL opted for a **parallel seq scan** across 2 worker processes because scanning large chunks linearly with multiple threads is faster than jumping around via index lookups.

3. **Planner Cost-Based Decision**
   PostgreSQL uses a **cost-based planner**, which means it evaluates different query plans and picks the one with the lowest estimated cost. In this case, even with an index, the sequential scan with parallelism was cheaper.

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