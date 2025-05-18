# Bitmap Index Scan Execution Flow
PostgreSQL uses Bitmap AND/OR operations to **combine multiple indexes** before accessing the heap. This reduces redundant I/O and fetches heap pages only once - leading to faster and more efficient execution for moderately large result sets.

## Flow: How PostgreSQL Executes Bitmap Index Scans

### Case 1: `BitmapOr` (e.g., `WHERE full_name = 'Employee_1245' OR salary = 5000`)

```
[Query Planner]
      |
      v
[Evaluate Conditions]
      |
      v
[Bitmap Index Scan on Index A] ----
                                   |
[Bitmap Index Scan on Index B] ----
        |
        v
     [BitmapOr]
        |
        v
 [Combined Bitmap with OR matches]
        |
        v
[Bitmap Heap Scan to fetch rows]
        |
        v
  [Final Result Set]
```

### Case 2: `BitmapAnd` (e.g., `WHERE full_name = 'Employee_1276' AND dept_id = 4`)

```
[Query Planner]
      |
      v
[Evaluate Conditions]
      |
      v
[Bitmap Index Scan on Index A] ----
                                   |
[Bitmap Index Scan on Index B] ----
        |
        v
     [BitmapAnd]
        |
        v
 [Combined Bitmap with AND matches]
        |
        v
[Bitmap Heap Scan to fetch rows]
        |
        v
  [Final Result Set]
```

## Notes

* **Bitmap Index Scans** are ideal when multiple index conditions are involved.
* They **combine row locations** from multiple indexes **before** hitting the heap.
* PostgreSQL uses:
  * `Bitmap AND` to **intersect** rows satisfying all conditions.
  * `Bitmap OR` to **union** rows satisfying any condition.
* Final heap access is **batched**, reducing I/O overhead.