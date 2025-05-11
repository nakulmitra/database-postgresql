# What Happens Internally in a Sequential Scan

1. **Start from the beginning of the table (heap)**.
2. PostgreSQL reads the table **block by block (page by page)**.
3. Each page is:
   * Fetched from disk **into shared buffers (memory)** if it's not already there.
   * Once in memory, PostgreSQL loops through all **tuples (rows)** in that page.
4. It **applies the WHERE clause (filter)** on each row.
5. If a row matches, it's included in the result set.
6. This continues until the **entire table** is scanned.

> A **heap page** in PostgreSQL is typically 8 KB and may contain multiple tuples.

## Why Load Into Memory?

* PostgreSQL **never reads rows directly from disk** without first **loading the corresponding page into memory (buffer cache)**.
* This memory-based processing allows it to:

  * Reuse pages already in memory
  * Maintain consistency via buffer management
  * Support concurrency and MVCC

So yes - **every row in a sequential scan comes from a page that is loaded into memory first**, not directly from disk.

## Internal Flow of Sequential Scan in PostgreSQL

```
[Start Sequential Scan]
        |
        v
[Access the heap (table) from beginning to end]
        |
        v
[Read each page (block) one by one from disk]
        |
        v
[Load page into shared buffers (memory) if not already in memory]
        |
        v
[Iterate over each tuple (row) in the page]
        |
        v
[Apply WHERE clause/filter condition]
        |
        |---> [Row matches condition] ---> [Include in result]
        |
        |---> [Row does not match] ------> [Skip]
        |
        v
[Repeat for all pages in the table]
        |
        v
[Return final result set]
```

## Notes:

* Each page is typically **8 KB** in size and may contain multiple rows.
* PostgreSQL uses a **buffer pool (shared memory)** to cache disk pages, improving performance if pages are accessed multiple times.
* Even if a row is not needed, its **entire page must be loaded** into memory for evaluation.
* This method is **fast for small tables** or **when most rows match**, but **inefficient for large tables with selective filters**.