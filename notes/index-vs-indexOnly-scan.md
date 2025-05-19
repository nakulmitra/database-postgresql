# PostgreSQL Index Scan vs Index-Only Scan

Optimizing query performance is essential for working with large datasets in PostgreSQL. Two closely related access methods play a vital role in speeding up SELECT queries: **Index Scan** and **Index-Only Scan**. Understanding their differences and how PostgreSQL chooses between them is key to efficient database design.

[![](https://markdown-videos-api.jorgenkh.no/youtube/ZxqfSDND5bg)](https://youtu.be/ZxqfSDND5bg)

## What is an Index Scan?

An **Index Scan** is a method PostgreSQL uses when it needs to:

* Locate matching rows **using an index**, and
* **Access the heap (table)** to retrieve either:

  * Columns that are not present in the index, or
  * **Visibility information** (to verify whether the row is visible to the current transaction due to MVCC).

### Characteristics:

* Index is used to identify **row locations (TIDs)**.
* The actual data is then fetched **from the table (heap)**.
* Even if all required columns are in the index, PostgreSQL may still hit the heap to **verify visibility**, unless the visibility map indicates the page is all-visible.

### Example:

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT full_name FROM tutorial.employees WHERE id = 189;
```

```text
Index Scan using idx_id on employees  
  Index Cond: (id = 189)
  Buffers: shared hit=5
```

> PostgreSQL uses the index to find rows matching `id = 189`, but still accesses the heap to retrieve `full_name` and verify row visibility.

## What is an Index-Only Scan?

An **Index-Only Scan** is an enhanced form of index usage where PostgreSQL:

* Satisfies the query **entirely from the index**, **without accessing the heap**.

### Conditions for Index-Only Scan:

1. **All columns** used in the query (SELECT/WHERE) are **present in the index**.
2. **Visibility Map** confirms that all required table pages are "all-visible".

If these conditions are met, PostgreSQL avoids the costly heap access.

### Example:

```sql
CREATE INDEX full_name_salary_idx ON tutorial.employees(full_name, salary);

EXPLAIN (ANALYZE, BUFFERS)
SELECT salary FROM tutorial.employees WHERE full_name = 'Employee_124';
```

```text
Index Only Scan using full_name_salary_idx on employees  
  Index Cond: (full_name = 'Employee_124')
  Heap Fetches: 1
```

> Although this is an index-only scan, a heap fetch is still required due to the visibility map not being fully up to date. We can reduce heap fetches by running `VACUUM ANALYZE`.

## Key Differences: Index Scan vs Index-Only Scan

| Feature                   | Index Scan        | Index-Only Scan                     |
| ------------------------- | ----------------- | ----------------------------------- |
| Accesses table heap?      | Yes               | No (if possible)                    |
| Uses visibility map?      | No                | Yes                                 |
| Columns required in index | Filter column     | Filter + SELECT columns             |
| Speed                     | Slower (more I/O) | Faster (less I/O)                   |
| Heap Fetches              | Always            | Only if page not marked all-visible |

## When PostgreSQL Chooses Index-Only Scan

PostgreSQL uses an Index-Only Scan **when all conditions are met**:

* All columns in the SELECT and WHERE clause are **included in the index**
* **Visibility map** marks relevant heap pages as all-visible
* Index has **good selectivity**

## How to Enable Index-Only Scans

To ensure PostgreSQL can choose an Index-Only Scan, follow these best practices:

### 1. Use Covering Indexes

Include all columns required by our queries in the index:

```sql
CREATE INDEX idx_covering ON employees(full_name, salary);
```

### 2. Keep Visibility Map Updated

Run regular `VACUUM ANALYZE` to mark pages as all-visible:

```sql
VACUUM ANALYZE employees;
```

### 3. Avoid Selecting Unneeded Columns

Only select columns that really need to avoid unnecessary heap access.

## Real-World Comparison

### Index-Only Scan:

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT full_name FROM tutorial.employees WHERE id = 167;
```

```text
Index Only Scan using idx_id on employees  
  Index Cond: (id = 167)
  Heap Fetches: 0
```

> No heap fetch needed. Visibility map is up to date. Excellent performance.

### Index Scan (fallback):

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT id FROM tutorial.employees WHERE full_name = 'Employee_123';
```

```text
Index Scan using idx_full_name_salary on employees  
  Index Cond: ((full_name)::text = 'Employee_123')
```

> Heap access was required, likely due to visibility map or missing index coverage.

### Sequential Scan fallback (non-matching index):

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT id FROM tutorial.employees WHERE salary = 234;
```

```text
Parallel Seq Scan on employees  
  Filter: (salary = '234')
```

> Index not useful because the filtering column wasn't first in the index or not selective.

## Summary

| Condition                          | PostgreSQL Action          |
| ---------------------------------- | -------------------------- |
| All columns in index + all-visible | Index-Only Scan            |
| Some columns missing from index    | Falls back to Index Scan   |
| Visibility map outdated            | Falls back to Index Scan   |
| Table is small / poor selectivity  | May choose Sequential Scan |

## Performance Tips

* Use **covering indexes** with needed columns.
* Keep tables **VACUUMED** to maintain visibility map.
* Validate execution plans with:

```sql
EXPLAIN (ANALYZE, BUFFERS) <query>;
```