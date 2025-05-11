# What is an Index in a Database?

## Overview
In databases, performance is everything - especially when dealing with large volumes of data. An **Index** is a powerful tool that allows databases to **quickly locate and access data** without scanning every row in a table.

[![](https://markdown-videos-api.jorgenkh.no/youtube/whHvqayU38c)](https://youtu.be/whHvqayU38c)

## What is an Index?
An **Index** is a **data structure** that helps the database **find rows faster**, similar to the index at the back of a book. Instead of reading through every page to find a topic, you go to the index and directly jump to the correct page.

In databases like **PostgreSQL**, indexes are typically implemented using a **B-Tree (Balanced Tree)**. This allows:
- **Logarithmic time complexity (O(log n))** for searches
- Efficient insertions and lookups  
- Sorted traversal of data

## Why Do We Need Indexes?

Without indexes, the database must **scan every row** to find matching records - known as a **sequential scan**. This becomes **very inefficient for large tables**.

### Indexes are used to:
- Improve performance of **SELECT queries**
- Speed up **JOINs**, **WHERE**, **ORDER BY**, **GROUP BY**
- Enforce **UNIQUE** and **PRIMARY KEY** constraints
- Support faster access paths for foreign keys

### But Indexes Also Have a Cost:
- **Additional storage space** is used
- **Insert/Update/Delete operations become slightly slower** (since the index must be updated)

## How Index Improves Query Performance (Internal Mechanics)

Let's break this down using PostgreSQL.

### Scenario 1: Without Index - Sequential Scan (Seq Scan)

```sql
SELECT * FROM employees WHERE full_name = 'dev';
```

#### Internal Steps:
1. `Heap Storage:` PostgreSQL stores table data in fixed-size **pages** (typically 8KB), called **heap pages**.
2. `Sequential Scan:` PostgreSQL reads **every page** one by one from disk. Inside each page, it scans each tuple (row) sequentially.
3. `Buffer Pool:` If the page isn't already in memory, it's loaded into PostgreSQL's **shared buffer pool**.
4. `Row Filtering:` After loading a row, it applies the filter (`full_name = 'dev'`).
5. `Return:` Only matching rows are returned to the client.

#### Performance:
- Time Complexity: **O(n)**
- High **disk I/O** and memory usage
- Slow for large datasets

#### Sequential Scan Execution Plan
![Sequential Scan Query Planner](https://github.com/nakulmitra/database-postgresql/blob/master/notes/images/Sequential_Scan_Query_Planner.png)

- PostgreSQL scanned **all 5 crore rows** (with parallel workers), even though the result had 1 matches.

```sql
explain analyze select * from demo.employees
where full_name = 'dev';
```

**Visual Output Summary:**
- **Execution Time**: ~17783 ms (~17.7 seconds)
- **Plan Type**: Parallel Sequential Scan
- **Rows Scanned**: 5 crore (50 million)

##### Step-by-Step Breakdown:

1. **Parallel Seq Scan on `employees`**:
   - PostgreSQL decided **not to use an index**.
   - It used a **parallel scan** to divide the work among 3 workers (main + 2).
   - Each worker scanned around 16.6 million rows.

2. **Filter**:
   ```
   Filter: ((full_name)::text = 'dev'::text)
   ```
   - The condition was applied **after** reading each row.
   - This is inefficient for selective queries, especially when only a few rows match.

3. **Rows Removed by Filter**:
   - **16,666,667 rows removed** per worker.
   - PostgreSQL had to scan every row in the heap.

4. **Execution Time**:
   - **Planning Time**: 3.933 ms
   - **Execution Time**: 17783.043 ms
   - This is about **250x slower** than the index scan.

5. **Why Seq Scan?**
   - In this case, it might have been executed **before** the index was created.
   - Or PostgreSQL's planner estimated that the cost of the index wasn't worth it (if data was too uniform or stats were outdated).

> To understand the flow of Sequential Scan, please refer the link:
> [Sequential Scan Flow Dig](https://github.com/nakulmitra/database-postgresql/blob/master/flowDig/seqScan.md)


### Scenario 2: With Index - Index Scan (and Heap Fetch)

```sql
CREATE INDEX idx_full_name ON employees(full_name);
SELECT * FROM employees WHERE full_name = 'dev';
```

#### Internal Steps:
1. `Index Structure:` The B-Tree stores `full_name` values in **sorted order**, along with **TIDs** (Tuple IDs: block number + offset).
2. `Index Lookup:` PostgreSQL traverses the B-Tree in **log(n)** time to find matching entries.
3. `Heap Fetch:` Using the TID (Tuple ID), it directly fetches the required **heap page** and **row offset**.
4. `MVCC Check:` Validates if the row is visible to the current transaction.
5. `Return:` Matching row is returned immediately.

#### Performance:
- Time Complexity: **O(log n) + O(1)**
- Only relevant disk pages and rows are touched
- Fast, especially for selective queries

#### Index Scan Execution Plan
![Index Scan Query Planner](https://github.com/nakulmitra/database-postgresql/blob/master/notes/images/Index_Scan_Query_Planner.png)

```sql
explain analyze select * from demo.employees
where full_name = 'dev';
```

**Visual Output Summary:**
- **Index Used**: `idx_full_name`
- **Execution Time**: ~0.072 ms
- **Plan Type**: Index Scan

##### Step-by-Step Breakdown:

1. **Index Scan using `idx_full_name`**:
   - PostgreSQL is using a B-Tree index on `full_name` (`idx_full_name`).
   - It quickly traverses the index (logarithmic time) to find the matching `full_name = 'dev'`.
   - This is much faster than scanning every row in the table.

2. **Index Condition**:
   ```
   Index Cond: ((full_name)::text = 'dev'::text)
   ```
   - PostgreSQL is only scanning index entries that match the value `'dev'`.
   - This is efficient and avoids reading unrelated rows.

3. **Heap Fetch**:
   - Since you're using `SELECT *`, PostgreSQL fetches the actual tuple from the heap using the TID (Tuple ID) from the index.
   - Even though it's an index scan, a heap fetch still occurs because not all required columns are in the index.

4. **Performance**:
   - **Planning Time**: 0.133 ms
   - **Execution Time**: 0.072 ms
   - **Actual Rows Returned**: 1
   - Very fast due to minimal disk I/O.

> To understand the flow of Index Scan, please refer the link:
> [Index Scan Flow Dig](https://github.com/nakulmitra/database-postgresql/blob/master/flowDig/indexScan.md)


### Side-by-Side Comparison

| Criteria                  | Index Scan                            | Sequential Scan                      |
|--------------------------|----------------------------------------|--------------------------------------|
| Query Plan               | Index Scan using `idx_full_name`      | Parallel Seq Scan                    |
| Data Scanned             | Only relevant index + heap pages      | All 5 crore rows                     |
| Workers                  | 1                                      | 3 (parallel execution)               |
| Rows Removed             | N/A                                    | 50 million - 1 = 49,999,999          |
| Planning Time            | 0.133 ms                               | 3.933 ms                             |
| Execution Time           | 0.072 ms                               | 17783.043 ms (~17.7 seconds)         |
| Performance              | Super fast                             | Very slow                            |

## How to Create Indexes
### Single Column Index
```sql
CREATE INDEX idx_full_name ON employees(full_name);
```

### Multi-Column Index
```sql
CREATE INDEX idx_dept_salary ON employees(dept_id, salary);
```

### Note:
- Multi-column indexes are **left-to-right dependent**.
- `WHERE dept_id = 10 AND salary > 50000` -> uses the index
- `WHERE salary > 50000` -> may not use the index

## Best Practices & Considerations

| Do | Avoid |
|------|----------|
| Use indexes on columns used in **WHERE, JOIN, ORDER BY** | Creating indexes on every column |
| Use `EXPLAIN ANALYZE` to verify index usage | Indexing low-selectivity columns (e.g., boolean flags) |
| Monitor index size and usage | Overusing multi-column indexes unnecessarily |
| Reindex or vacuum tables when needed | Forgetting the impact of indexes on writes |

## Key Concepts

| Term | Description |
|------|-------------|
| **Heap** | Area where actual table data is stored |
| **Page** | Fixed-size (8 KB) block for storage |
| **TID** | Tuple ID = (block number, row offset) |
| **Seq Scan** | Scans all rows/pages sequentially |
| **Index Scan** | Uses index to find TIDs, fetches heap rows |
| **Index-Only Scan** | Uses index only, skips heap access (if possible) |
| **B-Tree** | Balanced Tree structure used in most indexes |
| **Buffer Pool** | In-memory cache of disk pages |

## Conclusion
An index is one of the most powerful feature for **query performance optimization**. It dramatically reduces the time needed to find relevant rows in large tables. However, it comes with trade-offs like additional storage and slower writes.

Understanding how indexes work under the hood, especially in relational databases like PostgreSQL, and allows developers and DBAs to:
- Write better queries
- Choose optimal indexing strategies
- Design performant applications