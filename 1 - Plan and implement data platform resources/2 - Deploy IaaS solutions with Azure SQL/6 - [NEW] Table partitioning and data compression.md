## 🧩 **Why This Topic Matters (DP-300 skills)**

- Covers three skills: **Plan for table partitioning**, **Configure table partitioning**, **Configure data compression**.

---

## 🗂️ **Partitioning Building Blocks**

| Object                  | What it does                                                                                     |
| ----------------------- | ------------------------------------------------------------------------------------------------ |
| **Partition function**  | Maps values of the **partitioning column** to partition numbers using **boundary values**.       |
| **Partition scheme**    | Maps each partition to a **filegroup** (or `ALL TO ([PRIMARY])`).                                |
| **Partitioning column** | The **single** column that decides the partition. Computed columns must be **PERSISTED**.        |
| **Aligned index**       | Index on the **same (or equivalent) partition scheme** as the table. Needed for fast switching.  |

- **RANGE LEFT** (default): the boundary value belongs to the partition on its **left** (it is that partition's upper limit).
- **RANGE RIGHT**: the boundary value belongs to the partition on its **right** (it is that partition's lower limit). Use it for **date** columns, so midnight on the 1st of the month lands in that month's partition.
- **N boundary values = N + 1 partitions.**
- You **can't** partition on **timestamp**, LOB types (`varchar(max)`, `xml`, ...), CLR UDTs or alias types.
- Max **15,000 partitions** per table/index. Docs warn against **hundreds or thousands of partitions** unless strictly necessary (memory, DBCC and index rebuild cost).

---

## 🏗️ **Configure Partitioning (T-SQL)**

```sql
-- 1. Function: monthly boundaries, RANGE RIGHT for dates
CREATE PARTITION FUNCTION pfOrderDate (date)
AS RANGE RIGHT FOR VALUES ('2026-01-01', '2026-02-01', '2026-03-01');

-- 2. Scheme: Azure SQL Database only has PRIMARY -> use ALL TO
CREATE PARTITION SCHEME psOrderDate
AS PARTITION pfOrderDate ALL TO ([PRIMARY]);
-- On a VM / Managed Instance you could map to filegroups instead:
-- AS PARTITION pfOrderDate TO (FG_Old, FG_2026_01, FG_2026_02, FG_2026_03);

-- 3. Table: clustered key MUST include the partitioning column
CREATE TABLE dbo.Orders
(
    OrderID   bigint        NOT NULL,
    OrderDate date          NOT NULL,
    Amount    decimal(12,2) NOT NULL,
    CONSTRAINT PK_Orders PRIMARY KEY CLUSTERED (OrderDate, OrderID)
) ON psOrderDate (OrderDate);

-- 4. Allow lock escalation to partition level instead of table level
ALTER TABLE dbo.Orders SET (LOCK_ESCALATION = AUTO);

SELECT $PARTITION.pfOrderDate('2026-02-15') AS PartitionNumber;   -- which partition?
SELECT partition_number, rows
FROM sys.partitions
WHERE object_id = OBJECT_ID('dbo.Orders') AND index_id = 1;
```

- **Unique** clustered or nonclustered index: you **must** add the partitioning column to the key yourself.
- **Non-unique** index: the engine adds the partitioning column for you (as a key for clustered, as an included column for nonclustered), so the index stays **aligned**.
- A **nonaligned** index (different scheme) is allowed, but it **blocks partition switching** and `TRUNCATE ... WITH (PARTITIONS)`.

---

## 📍 **Where Partitioning Is Supported**

| Platform                     | Supported?                    | Filegroups                                                       |
| ---------------------------- | ----------------------------- | ---------------------------------------------------------------- |
| SQL Server on Azure VM       | ✅ All editions since 2016 SP1 | Multiple filegroups: tiered storage, piecemeal backup/restore    |
| Azure SQL Managed Instance   | ✅ All service tiers           | Multiple filegroups allowed (`ALTER DATABASE ... ADD FILEGROUP`) |
| Azure SQL Database           | ✅ All service tiers           | **PRIMARY only**: every partition must go on `[PRIMARY]`         |
| SQL database in Fabric       | ✅                             | PRIMARY only. **Switch partition and partition compression are blocked** (mirroring) |

- Multiple filegroups only help with **tiered storage** or **filegroup-level backup/restore**. Otherwise the docs **recommend a single filegroup**. Rows with a **NULL** partitioning value go to the **left-most** partition.

---

## 🔀 **Partition Switching & the Sliding Window**

- `ALTER TABLE ... SWITCH` moves a whole partition **as a metadata-only change** (near-instant, no data copied).
- Rules:
  - The target table or partition **must exist and be empty**.
  - Source and target must be in the **same filegroup**, with **identical columns and indexes**, and the indexes must be **aligned**.
  - **Data compression must match** between the source partition and the target.
- **Sliding window** pattern (e.g., keep 12 months):
  1. `SPLIT RANGE` to add an empty partition for the new month (set `NEXT USED` first).
  2. `SWITCH OUT` the oldest partition to an archive/staging table.
  3. `MERGE RANGE` to remove the now-empty boundary.

```sql
-- Add next month (VM/MI: choose the filegroup the new partition uses)
ALTER PARTITION SCHEME psOrderDate NEXT USED [PRIMARY];
ALTER PARTITION FUNCTION pfOrderDate() SPLIT RANGE ('2026-04-01');

-- Switch the oldest partition out to an empty table with identical structure
ALTER TABLE dbo.Orders SWITCH PARTITION 1 TO dbo.Orders_Archive
WITH (WAIT_AT_LOW_PRIORITY (MAX_DURATION = 5 MINUTES, ABORT_AFTER_WAIT = SELF));

-- Remove the old boundary
ALTER PARTITION FUNCTION pfOrderDate() MERGE RANGE ('2026-01-01');

-- Or just delete partitions with minimal logging (SQL Server 2016+, Azure SQL DB/MI)
TRUNCATE TABLE dbo.Orders WITH (PARTITIONS (1, 3 TO 4));
```

- `TRUNCATE ... WITH (PARTITIONS)` needs **aligned** indexes. Always **split/merge empty partitions**; otherwise data is moved and heavily logged.

---

## 🎯 **Partition Elimination & Performance Reality**

- **Partition elimination**: when the `WHERE` clause filters on the **partitioning column**, the optimizer **skips** partitions it doesn't need.
- Queries that **don't** filter on the partitioning column must touch **every partition**. A single-row seek can get **slower**, because there is one seek per partition.
- Docs: partitioning **rarely improves OLTP performance**. Its main value is **manageability**: fast load/archive, per-partition rebuild/compress, partition-level lock escalation.
- Joins are fast when tables are **collocated** (partitioned on the join column with equivalent functions).
- Partitioned index rebuilds use **sampled** statistics. Run `UPDATE STATISTICS ... WITH FULLSCAN` if you need full-scan stats.

---

## 🗜️ **Data Compression Types**

| Type                    | Applies to             | How it works / use case                                                                                     |
| ----------------------- | ---------------------- | ----------------------------------------------------------------------------------------------------------- |
| **ROW**                 | Rowstore heap/index    | Stores fixed-length types as variable length. Low CPU cost; good default for OLTP.                         |
| **PAGE**                | Rowstore heap/index    | ROW + **prefix + dictionary** compression. Bigger savings, more CPU. Good for read-heavy or large tables. |
| **COLUMNSTORE**         | Columnstore indexes    | Always on for columnstore; you can't turn it off. Analytics/HTAP.                                           |
| **COLUMNSTORE_ARCHIVE** | Columnstore indexes    | Extra **XPRESS** compression. Smallest size, slower queries. Use for **cold/historical** partitions.        |
| **XML compression**     | `xml` columns/indexes  | **SQL Server 2022+**, Azure SQL Database and Managed Instance. Compresses **off-row XML** data.            |

- Compression can be set **per partition**, e.g. PAGE for old months and ROW for the current month.
- **Nonclustered indexes don't inherit** the table's compression. Set it on each index explicitly.
- **Off-row / LOB data isn't compressed** by ROW/PAGE. Use `COMPRESS()` (GZIP) or XML compression for that.
- A heap set to PAGE only page-compresses on **bulk load, `INSERT ... WITH (TABLOCK)`, or rebuild**.
- Changes can run **ONLINE**. The space needed is the same as an index rebuild, so compress **one partition at a time** to limit space use.
- Columnstore in Azure SQL Database DTU tiers needs **Standard S3 or higher** (or Premium/vCore).

```sql
-- 1. Estimate first (samples the data into tempdb)
EXEC sys.sp_estimate_data_compression_savings
     @schema_name = 'dbo', @object_name = 'Orders',
     @index_id = NULL, @partition_number = NULL,
     @data_compression = 'PAGE';
-- SQL 2022+/Azure SQL: optional 6th parameter @xml_compression = 1

-- 2. Apply: different compression per partition
ALTER TABLE dbo.Orders REBUILD PARTITION = ALL
WITH (DATA_COMPRESSION = PAGE ON PARTITIONS (1 TO 3),
      DATA_COMPRESSION = ROW  ON PARTITIONS (4), ONLINE = ON);

ALTER INDEX IX_Orders_Amount ON dbo.Orders REBUILD WITH (DATA_COMPRESSION = ROW);
ALTER TABLE dbo.Sales_CCI REBUILD PARTITION = 1 WITH (DATA_COMPRESSION = COLUMNSTORE_ARCHIVE);
ALTER TABLE dbo.Docs REBUILD WITH (XML_COMPRESSION = ON);

-- 3. Verify
SELECT OBJECT_NAME(object_id) AS tbl, index_id, partition_number, data_compression_desc
FROM sys.partitions WHERE object_id = OBJECT_ID('dbo.Orders');
```

- If the estimate shows the size **growing**, don't compress. This happens when the data already uses almost the full precision of its types.
- **Split** → both partitions inherit the original's compression; **merge** → the result takes the **destination** partition's setting. Not compatible with **sparse columns**.

---

## ✅ **Summary**

- **Function** (boundaries, LEFT/RIGHT) → **scheme** (filegroups) → **table/index ON scheme(column)**.
- Keep indexes **aligned** to allow `SWITCH` and `TRUNCATE ... WITH (PARTITIONS)`.
- Azure SQL Database = **PRIMARY filegroup only**. MI and VMs can use multiple filegroups.
- Use **ROW** for OLTP, **PAGE** for large read-mostly data, **COLUMNSTORE_ARCHIVE** for cold columnstore partitions, **XML compression** for big XML (2022+/Azure).
- Always run `sp_estimate_data_compression_savings` **before** compressing.

---

## 🎯 **Exam Tips / Traps**

- "Remove a month of old data **instantly with minimal logging**" → **SWITCH OUT** the partition (or `TRUNCATE ... WITH (PARTITIONS)`), not `DELETE`.
- "Archive old partitions to **cheaper storage**" → separate **filegroups** on a **VM/MI**. This is **not possible in Azure SQL Database** (PRIMARY only).
- "Reduce lock contention on a partitioned table" → `LOCK_ESCALATION = AUTO`.
- Switching fails? Check that the target is **empty**, the **filegroup** is the same, indexes are **aligned**, and **compression** matches.
- "Which compression for rarely queried **historical columnstore** data?" → **COLUMNSTORE_ARCHIVE**.
- "Estimate savings **before** enabling compression" → `sp_estimate_data_compression_savings`.
- Partitioning ≠ automatic speed-up. Without partition elimination, queries can get **slower**.
- A **unique** index on a partitioned table must **include the partitioning column**, or it becomes nonaligned.

> Source: https://learn.microsoft.com/en-us/sql/relational-databases/partitions/partitioned-tables-and-indexes · https://learn.microsoft.com/en-us/sql/relational-databases/data-compression/data-compression · https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-estimate-data-compression-savings-transact-sql · https://learn.microsoft.com/en-us/sql/t-sql/statements/truncate-table-transact-sql · https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-table-transact-sql · https://learn.microsoft.com/en-us/fabric/database/sql/limitations
