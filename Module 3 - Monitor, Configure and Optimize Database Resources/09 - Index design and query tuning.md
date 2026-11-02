# Index design and query tuning

> **Exam skills:** Identify and implement index changes for queries, Recommend query construct modifications based on resource usage
> **Reading time:** ~12 min

## The big idea
Most slow queries are fixed by a **better index** or a **better-written query**. Good table design (right data types, the right level of normalization) helps too. Query hints are the last resort. Always check that a change helps the **whole system**, not just one query.

Think of a phone book. It is sorted by surname (clustered index), so finding "Smith" is fast. Finding everyone in "Leeds" means reading every page, unless you add a second list sorted by city (nonclustered index).

## Index types
### Clustered index
- Sorts and stores the **table rows themselves** by the key. Only **one** per table.
- A table with no clustered index is a **heap**.
- Good keys are **narrow, unique, and ever-increasing**, for example an `INT IDENTITY` ID.

### Nonclustered index
- A **separate structure** with the key columns and a pointer to the row. Many per table.
- If the query needs columns not in the index, SQL Server does a **Key Lookup** back to the table.
- A **covering index** holds every column the query needs, using `INCLUDE`:

```sql
CREATE NONCLUSTERED INDEX IX_Products_ProductID
ON dbo.Products (ProductID) INCLUDE (ProductName, Price);
```

### Filtered index
An index on **some rows only**, using `WHERE`. Smaller and cheaper to maintain.
```sql
CREATE NONCLUSTERED INDEX IX_Employees_Current
ON HR.Employee (BusinessEntityID) WHERE CurrentFlag = 1;
```

### Columnstore index
- Stores data **by column** and compresses it. Reads only the columns needed.
- Best for **analytics and data warehouses** (big `SUM`, `AVG`, `GROUP BY`). Uses **batch mode** (about 900 rows at a time).
- Small loads (under 102,400 rows) go first to a row-based **delta store**; the **tuple mover** compresses it later. Force it with `ALTER INDEX ... REORGANIZE WITH (COMPRESS_ALL_ROW_GROUPS = ON)`.
- **Ordered columnstore** (`... ORDER (col)`) sorts data before compressing for better segment skipping. SQL Server 2022+, Azure SQL DB and MI.

```sql
CREATE CLUSTERED COLUMNSTORE INDEX CCI_FactSales ON dbo.FactSales;
```

## Designing and tuning indexes
- Index columns used in **WHERE**, **JOIN** and **ORDER BY**, especially foreign keys.
- **Column order:** put **equality** columns (`=`) first, then **range** columns (`>`, `<`, `BETWEEN`).

```sql
-- WHERE CustomerID = 10 AND OrderDate > '2025-01-01'
CREATE INDEX IX_Orders_Cust_Date ON dbo.Orders (CustomerID, OrderDate);
```
- Keep keys **small**. Integers compare faster than long strings.
- **Every index slows writes.** INSERT, UPDATE and DELETE must change each index. Use as few indexes as possible; widen an existing one with INCLUDE rather than adding another.
- **Last-page insert contention** (`PAGELATCH_EX` on an ever-increasing key like IDENTITY under heavy inserts): use `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON` (SQL Server 2019+, Azure SQL; default OFF).

### Finding index changes
| Need | Where to look |
|------|---------------|
| Missing indexes | Green hint in the plan, `sys.dm_db_missing_index_details`, automatic tuning, Intelligent Insights |
| How indexes are used | `sys.dm_db_index_usage_stats` (seeks, scans, lookups, updates) |
| Low-level index activity | `sys.dm_db_index_operational_stats` |
| Expensive queries to index for | Query Store, Extended Events |

```sql
SELECT OBJECT_NAME(i.object_id) AS tbl, i.name, u.user_seeks, u.user_scans, u.user_lookups, u.user_updates
FROM sys.indexes AS i
JOIN sys.dm_db_index_usage_stats AS u
  ON i.object_id = u.object_id AND i.index_id = u.index_id;
```
- **Few seeks/scans but many updates** → the index may cost more than it helps.
- Do **not** drop an index just because it is rarely used. It may be needed for month-end or year-end reports. One option: create it just before that report runs.
- Test new indexes outside production first and compare plans **before and after**.

**Example:** A school's "find student by email" query scans 2 million rows. `sys.dm_db_missing_index_details` suggests an index on `Email` including `FullName`. After creating it, the plan shows an Index Seek and reads drop from 40,000 pages to 4.

## Table design for performance
### Normalization
**Normalization** = "one fact in one place". It removes repeated data and update mistakes.
- **1NF:** no repeating groups (not `Color1, Color2, Color3`; one row per colour). Each row has a primary key.
- **2NF:** with a composite key, every non-key column depends on the **whole** key.
- **3NF:** no column depends on another **non-key** column (move Country code to a Countries table).

| Workload | Design |
|----------|--------|
| **OLTP** (many small reads and writes, e.g. a shop checkout) | **Normalize**, index key columns, short transactions |
| **OLAP / data warehouse** (big reports) | **Denormalize** to reduce joins; **star schema** (one fact table + dimension tables) or **snowflake schema** (dimensions split into more tables: less repeated data, more joins); columnstore; partitioning |

**Partitioning** splits a big table into parts, for example Sales by year, so a query for one year reads only that part.

### Data types
- Use the **smallest correct type**. Smaller rows = more rows per page = less I/O and memory.
- Numbers in numeric types (`INT`, `DECIMAL(10,2)` for money), dates in `DATE` / `DATETIME2`, not in `VARCHAR`.
- `UNIQUEIDENTIFIER` is 16 bytes; use only when needed. Native **JSON** and **VECTOR** types now exist in Azure SQL DB, MI (newer update policies) and SQL Server 2025.

### Implicit conversion
When two different types are compared, SQL Server converts one silently. The **lower-precedence type is converted** to the higher one. INT beats VARCHAR.

```sql
-- PhoneNumber is VARCHAR. The COLUMN is converted to INT on every row -> no seek
SELECT * FROM dbo.Customers WHERE PhoneNumber = 12345;
-- Better: compare with the same type
SELECT * FROM dbo.Customers WHERE PhoneNumber = '12345';
```
The plan shows a **warning** for implicit conversions. Fix by matching the parameter type to the column type.

## Rewriting queries based on resource usage
A **SARGable** query (Search ARGument-able) lets SQL Server use an index seek. Wrapping the column in a function or calculation stops this.

| Bad (scan) | Good (seek possible) |
|------------|----------------------|
| `WHERE LEFT(City, 1) = 'M'` | `WHERE City LIKE 'M%'` |
| `WHERE YEAR(OrderDate) = 2025` | `WHERE OrderDate >= '2025-01-01' AND OrderDate < '2026-01-01'` |
| `WHERE CAST(OrderID AS VARCHAR(10)) = '100'` | `WHERE OrderID = 100` |

Other rewrites:
| Pattern | Problem | Better |
|---------|---------|--------|
| Cursors / `WHILE` loops | Row-by-row work, high CPU | **Set-based** `UPDATE` / `INSERT ... SELECT` |
| Multi-statement table-valued functions | Bad row estimates | **Inline** TVFs (or rely on interleaved execution) |
| Scalar functions on every row | Slow, no parallelism | Inline the logic (or compat 150+ scalar UDF inlining) |
| Huge single UPDATE/DELETE | Lock escalation, big log | Smaller batches |

**Match the fix to the resource:**
- **High CPU** → tune top CPU queries, remove row-by-row code, add indexes, check parallelism.
- **High I/O / PAGEIOLATCH** → indexes to stop scans, covering indexes, fewer columns.
- **Memory grants / spills** → fix estimates (statistics), avoid huge sorts.
- **Log rate limit** → smaller batches, fewer index writes.

## Parameter sniffing
SQL Server builds a plan using the **first parameter value** it sees, then reuses that plan. If the first value was unusual, the plan may be bad for later values.

**Example:** `EXEC GetOrders @CustomerID = 42` (customer 42 has 10 million orders) builds a scan plan. Later `@CustomerID = 5` (10 orders) reuses the scan plan and is slow.

| Fix | Trade-off |
|-----|-----------|
| `OPTION (RECOMPILE)` | New plan each run; more CPU |
| `OPTIMIZE FOR (@p = value)` | Plan built for a typical value |
| `OPTIMIZE FOR UNKNOWN` | Uses average statistics, ignores the actual value |
| Recompile only for unusual values (IF branch) | More code |
| **PSP optimization** (compat 160) | Keeps several plans automatically |
| Force a good plan in Query Store | Plan fixed until unforced |

## Query hints
Hints force the optimizer's choice. Add them with `OPTION (...)`.
| Hint | Effect |
|------|--------|
| `MAXDOP n` | Limit cores for this query |
| `RECOMPILE` | Build a new plan every run |
| `OPTIMIZE FOR (@p = x)` / `OPTIMIZE FOR UNKNOWN` | Choose which value to optimize for |
| `FAST n` | Return the first n rows quickly |
| `LOOP JOIN` / `MERGE JOIN` / `HASH JOIN` | Force a join type |
| `USE PLAN N'<xml>'` | Force a given XML plan |

```sql
SELECT ProductID, SUM(LineTotal)
FROM Sales.SalesOrderDetail
WHERE UnitPrice < 5.00
GROUP BY ProductID
OPTION (MAXDOP 2, RECOMPILE);
```
- Use hints **rarely**: for urgent fixes or code you cannot change. They stop the optimizer from adapting to new data, new indexes and new versions.
- Fix the **root cause** first (index, statistics, query text).
- If you cannot change the code, use **Query Store hints** (lesson 7).

## Check the whole system
A change that helps one query can hurt others. A new index speeds up a `SELECT` but slows `INSERT`s on that table. Compare plans, Query Store, wait stats, CPU and I/O **before and after**, and test on a **staging** server with a similar workload first.

## Remember for the exam
- One **clustered** index per table; many **nonclustered**. No clustered = **heap**.
- **Key Lookup** → covering index with **INCLUDE**.
- Small subset of rows → **filtered index**. Big analytics → **columnstore**.
- Index column order: **equality first, then range**.
- Too many indexes slow writes. Check `sys.dm_db_index_usage_stats` before dropping; rare ≠ useless.
- Functions or conversions on a column in WHERE → **non-SARGable** → scan. Rewrite.
- **Implicit conversion:** the lower-precedence side is converted; VARCHAR column vs INT value converts the column.
- OLTP → normalize. OLAP → denormalize, star schema, columnstore.
- Parameter sniffing → RECOMPILE, OPTIMIZE FOR (UNKNOWN), PSP, or force a plan.
- Hints are a last resort; Query Store hints when the code cannot change.

## Quick check
1. Query: `WHERE Status = 'Open'`, and only 2% of 50 million rows are Open. Which index is smallest and best?
   <details><summary>Answer</summary>A filtered nonclustered index with WHERE Status = 'Open'. It only stores the Open rows.</details>
2. `WHERE YEAR(OrderDate) = 2025` scans the table even though OrderDate has an index. How do you fix it?
   <details><summary>Answer</summary>Make it SARGable: WHERE OrderDate >= '2025-01-01' AND OrderDate < '2026-01-01'. The function on the column blocked the seek.</details>
3. A query is fast for most customers but slow for one very large customer, because a cached plan is reused. What is this, and name one fix?
   <details><summary>Answer</summary>Parameter sniffing. Fixes: OPTION (RECOMPILE), OPTIMIZE FOR UNKNOWN, PSP optimization at compat 160, or forcing a suitable plan.</details>
