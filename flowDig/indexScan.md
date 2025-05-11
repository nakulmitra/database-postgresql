# Index Scan

PostgreSQL uses an index to find matching rows and then fetches each corresponding row from the heap.
This reduces the number of scanned rows but may involve extra I/O due to heap lookups.

## Internal Flow of Index Scan in PostgreSQL

```
[Start Index Scan]
        |
        v
[Access the index structure (e.g., B-Tree)]
        |
        v
[Traverse index to find matching entries for WHERE clause]
        |
        v
[For each matching index entry, get the TID (tuple ID)]
        |
        v
[Use TID to fetch the corresponding row from the heap (main table)]
        |
        v
[Load heap page into shared buffers (memory) if not already loaded]
        |
        v
[Fetch the row from the heap]
        |
        v
[Check visibility using MVCC (Multi-Version Concurrency Control)]
        |
        v
[Return row if visible and matches condition]
        |
        v
[Repeat for all matching index entries]
        |
        v
[Return final result set]
```

## Notes:

* Index scan reads fewer pages than a sequential scan when the filter is selective.
* **Heap access adds extra I/O**, especially if many scattered rows must be fetched.
* MVCC check ensures that the row is visible to the current transaction snapshot.