# Index-Only Scan

PostgreSQL uses the index alone to satisfy the query, without accessing the heap, if all needed columns are in the index.
It is the fastest scan type but only possible when visibility and column coverage conditions are met.

## Internal Flow of Index-Only Scan in PostgreSQL

```
[Start Index-Only Scan]
        |
        v
[Access the index structure (e.g., B-Tree)]
        |
        v
[Traverse index to find matching entries for WHERE clause]
        |
        v
[Check if all required columns can be fetched directly from the index]
        |
        v
[For each matching entry, read data from the index (no heap access)]
        |
        v
[Check visibility map to ensure the row is visible]
        |
        |---> [Row is visible in visibility map] ---> [Return from index]
        |
        |---> [Row is not visible] ---> [Fall back to heap fetch + MVCC check]
        |
        v
[Repeat for all matching index entries]
        |
        v
[Return final result set]
```

## Notes:

* This scan is **faster than Index Scan** as it avoids heap fetches when possible.
* Requires a **covering index** (index contains all selected columns).
* Relies on the **visibility map**, which PostgreSQL maintains during vacuuming.