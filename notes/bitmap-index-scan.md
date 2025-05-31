# Bitmap Index Scan in PostgreSQL

PostgreSQL offers multiple access methods to retrieve rows efficiently based on the query and data distribution. One such powerful and often misunderstood mechanism is the **Bitmap Index Scan**.

## What is a Bitmap Index Scan?

A **Bitmap Index Scan** is an advanced query execution strategy used by PostgreSQL when:

* Multiple **conditions** exist, each with its own index
* **Moderate number of rows** match - not too few (for Index Scan) and not too many (for Seq Scan)
* PostgreSQL wants to **reduce heap access** by batching reads

Instead of fetching each matching row individually (as with a regular Index Scan), PostgreSQL:

1. Scans the index and creates a **bitmap** of matching row locations.
2. Combines multiple bitmaps using **Bitmap AND / Bitmap OR**.
3. Fetches heap pages **in batch**, reducing random I/O and improving performance.

## Bitmap Index Scan vs Index Scan

| Feature                | Index Scan             | Bitmap Index Scan                     |
| ---------------------- | ---------------------- | ------------------------------------- |
| Access Method          | Row-by-row via index   | Bitmap of matches -> batch heap access |
| Heap Access            | Immediate              | Deferred + combined                   |
| Use Case               | Very selective filters | Moderately selective filters          |
| Multi-index support    | No                   | Yes (via Bitmap AND / Bitmap OR)    |
| Heap page optimization | No                   | Yes                                 |

## Real Query Example

Assume we have a table `employees` with **50 million** rows. We want to search for employees by `full_name` or `salary`.

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM tutorial.employees
WHERE full_name = 'Employee_234' OR salary = 70064.00;
```

PostgreSQL uses:

* Bitmap Index Scan on `full_name`
* Bitmap Index Scan on `salary`
* Combines them with **BitmapOr**
* Then fetches rows via **Bitmap Heap Scan**

## Query Plan Breakdown

![Bitmap Scan Query Planner](https://github.com/nakulmitra/database-postgresql/blob/master/notes/images/BitMap_Index_Scan.png)

### Breakdown:

#### Step 1: Bitmap Index Scan on `full_name`

* Index: `idx_full_name_salary`
* Matching TIDs are recorded in a bitmap
* Shared buffers hit = 4 (in memory)

#### Step 2: Bitmap Index Scan on `salary`

* Index: `idx_salary`
* Another bitmap created
* Buffers read = 5 (from disk), hit = 1

#### Step 3: BitmapOr

* Both bitmaps are **merged using OR**
* Buffers: 5 pages (combined)

#### Step 4: Bitmap Heap Scan

* PostgreSQL reads heap blocks in **batch**
* Avoids repeated access to the same block
* Matches rechecked for MVCC safety
* Heap blocks touched = 864

### Performance Stats:

* **Planning Time:** 0.144 ms
* **Execution Time:** 35.420 ms
* This is significantly faster than individual row-by-row index scans for high-match conditions

## When Does PostgreSQL Use Bitmap Index Scans?

PostgreSQL automatically switches to Bitmap Index Scan when:

* Filters are **not too selective** (10-15% of table)
* **Multiple conditions** can be supported by separate indexes
* **Random I/O** can be minimized through batching
* The **cost estimate** of Bitmap Heap Scan is lower than other strategies

## Performance Insights

* Works best for **medium-selectivity** filters
* Efficient when using **multiple conditions**
* Reduces **heap page fetches** by batching
* Not ideal for very **selective** queries (use Index Scan)
* Not used for very **broad** queries (Sequential Scan is better)

## Summary

Bitmap Index Scans are a **smart hybrid mechanism** in PostgreSQL. They combine the **precision of Index Scans** and the **efficiency of Sequential Scans** through bitmap logic and batch heap access.

### Key Advantages:

* Combines multiple indexes using `Bitmap AND` / `Bitmap OR`
* Minimizes heap fetches
* Efficient for complex filter conditions
* Great for **analytics, reporting**, and **multi-column search**