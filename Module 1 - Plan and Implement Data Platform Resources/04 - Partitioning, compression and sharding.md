# Partitioning, compression and sharding

> **Exam skills:** Plan for table partitioning, Configure table partitioning, Configure data compression, Recommend a database sharding solution
> **Reading time:** ~13 min

## The big idea
**Partitioning** cuts one big table into slices inside **one** database. **Compression** makes the data smaller on disk. **Sharding** spreads rows over **many** databases.

Analogy: a library. Partitioning = one shelf per year inside the same library. Compression = smaller print so more books fit. Sharding = opening branch libraries in other towns, each holding some of the books.

## Partitioning: the building blocks
| Object | What it does |
|--------|--------------|
| **Partition function** | Sets the **boundary values**. Decides which partition a value goes to. |
| **Partition scheme** | Maps each partition to a **filegroup** (a group of data files). |
| **Partitioning column** | The **one** column that decides the partition (for example `OrderDate`). |
| **Aligned index** | An index on the same partition scheme as the table. Needed for fast switching. |

- **RANGE LEFT** (default): the boundary value belongs to the partition on its **left**.
- **RANGE RIGHT**: the boundary value belongs to the partition on its **right**. Use it for **dates**, so 1 March lands in the March partition.
- **N boundary values = N + 1 partitions.**
- You can't partition on **timestamp**, LOB types (`varchar(max)`, `xml`), CLR types or alias types.
- Max **15,000 partitions** per table. Don't create thousands without a reason.
- Rows with a **NULL** partitioning value go to the left-most partition.

## Configure partitioning
```sql
CREATE PARTITION FUNCTION pfOrderDate (date)
AS RANGE RIGHT FOR VALUES ('2026-01-01', '2026-02-01', '2026-03-01');

CREATE PARTITION SCHEME psOrderDate
AS PARTITION pfOrderDate ALL TO ([PRIMARY]);   -- Azure SQL Database: PRIMARY only

CREATE TABLE dbo.Orders (OrderID bigint NOT NULL, OrderDate date NOT NULL,
  CONSTRAINT PK_Orders PRIMARY KEY CLUSTERED (OrderDate, OrderID)
) ON psOrderDate (OrderDate);
```
- A **unique** index must **include the partitioning column** in its key, or it is not aligned.
- For a **non-unique** index, SQL Server adds the column for you.
- `ALTER TABLE dbo.Orders SET (LOCK_ESCALATION = AUTO);` lets locks escalate to one partition, not the whole table.
- `SELECT $PARTITION.pfOrderDate('2026-02-15');` tells you which partition a value goes to.

## Where partitioning works
| Platform | Filegroups |
|----------|-----------|
| SQL Server on VM | Many filegroups. Old data can sit on cheaper disks. Filegroup backup/restore. |
| Managed Instance | Many filegroups allowed |
| SQL Database | **PRIMARY only** |
| SQL database in Fabric | PRIMARY only. Switch and per-partition compression are **blocked** (mirroring) |

Use many filegroups only for tiered storage or filegroup backups. Otherwise one filegroup is recommended.

## Partition switching and the sliding window
`ALTER TABLE ... SWITCH` moves a whole partition in or out as a **metadata-only** change. It is almost instant. No rows are copied.

Rules for SWITCH:
- The target table or partition must **exist and be empty**.
- Same **filegroup**, same columns and indexes, indexes **aligned**.
- **Compression must match.**

**Sliding window** (keep the last 12 months):
1. `SPLIT RANGE` to add an empty partition for next month (set `NEXT USED` first).
2. `SWITCH` the oldest partition out to an archive table.
3. `MERGE RANGE` to remove the empty old boundary.

```sql
ALTER PARTITION SCHEME psOrderDate NEXT USED [PRIMARY];
ALTER PARTITION FUNCTION pfOrderDate() SPLIT RANGE ('2026-04-01');
ALTER TABLE dbo.Orders SWITCH PARTITION 1 TO dbo.Orders_Archive;
ALTER PARTITION FUNCTION pfOrderDate() MERGE RANGE ('2026-01-01');
TRUNCATE TABLE dbo.Orders WITH (PARTITIONS (1));   -- or just empty a partition
```
Always split and merge **empty** partitions. Otherwise rows move and are heavily logged.

**Example:** An online shop keeps 12 months of orders. On the 1st of each month it switches out the oldest month in seconds, instead of a slow `DELETE` of 10 million rows.

## Does partitioning make queries faster?
- **Partition elimination:** if the `WHERE` filters on the partitioning column, SQL skips the other partitions.
- If the query does **not** filter on that column, it touches every partition. It can get **slower**.
- Microsoft says partitioning **rarely speeds up OLTP**. Its main value is **manageability**: fast load and archive, per-partition rebuild and compression, partition-level locks.

## Data compression types
| Type | Works on | Use |
|------|----------|-----|
| **ROW** | Rowstore tables and indexes | Stores fixed-length types in less space. Low CPU. Good OLTP default. |
| **PAGE** | Rowstore | ROW + **prefix + dictionary** compression. Bigger saving, more CPU. Large, read-heavy tables. |
| **COLUMNSTORE** | Columnstore indexes | Always on. Analytics. |
| **COLUMNSTORE_ARCHIVE** | Columnstore indexes | Smallest size, slower reads. **Cold, historical** data. |
| **XML compression** | `xml` columns | SQL Server 2022+, SQL Database, MI. |

Key rules:
- You can set compression **per partition** (PAGE for old months, ROW for this month).
- **Nonclustered indexes don't inherit** the table's compression. Set each one.
- ROW/PAGE do **not** compress off-row LOB data. Use `COMPRESS()` or XML compression.
- A heap set to PAGE only compresses new pages on bulk load, `INSERT ... WITH (TABLOCK)` or rebuild.
- You can rebuild with `ONLINE = ON`. Do one partition at a time to limit extra space.
- Columnstore in DTU tiers needs **Standard S3 or higher**.
- After SPLIT, both partitions keep the old setting. After MERGE, the result takes the setting of the partition it merges into.

```sql
EXEC sys.sp_estimate_data_compression_savings
     @schema_name = 'dbo', @object_name = 'Orders',
     @index_id = NULL, @partition_number = NULL, @data_compression = 'PAGE';

ALTER TABLE dbo.Orders REBUILD PARTITION = ALL
WITH (DATA_COMPRESSION = PAGE ON PARTITIONS (1 TO 3),
      DATA_COMPRESSION = ROW  ON PARTITIONS (4), ONLINE = ON);
```
Always **estimate first** with `sp_estimate_data_compression_savings`. If the estimate grows, don't compress.

**Example:** A bank's 5-year transaction history is rarely read. Its columnstore partitions for old years use COLUMNSTORE_ARCHIVE to save space.

## Sharding: what it is
**Sharding** is **scale out**: rows of the same table design are spread over many separate databases (**shards**). The **shard key** (often `TenantID` or `CustomerID`) decides where each row lives.
- **Scale up** = a bigger single database.
- **Scale out** = more databases.

## Try simpler options first
| Need | Best fit |
|------|----------|
| Database outgrowing 4 TB | **Hyperscale** (128 TB), no app changes |
| More read capacity | Read scale-out or Hyperscale named replicas |
| Many small databases with uneven load | Elastic pool |
| Throughput or size beyond any single database | **Sharding** |
| Tenants need physical isolation | Database per tenant (a form of sharding) |
| Data must stay in different countries | Shard by region |

Sharding adds work: routing, cross-shard queries, rebalancing, schema changes on N databases. Recommend it only when needed. If you sharded only because of size, think about moving into **Hyperscale**.

## Choosing a shard key
- In **every** sharded table, ideally the **first column of the primary key**.
- **High cardinality** (many distinct values) and **even spread**, so no shard is "hot".
- Matches the **most common query**, so most queries hit one shard.
- **Stable.** Changing it means moving the row.
- Supported types in the Elastic Database client library: int, bigint, GUID, varbinary, datetime, timespan, datetimeoffset.

## Sharding strategies
| Strategy | How | Good | Bad |
|----------|-----|------|-----|
| **Range** | Value ranges (IDs 1–1M → Shard A) | Simple; range queries stay on one shard | **Hot spots** if new data always lands in the newest range |
| **Hash** | Hash of the key (`UserID % 4`) | Even load, no hot spots | **Range queries are hard**, data is scattered |
| **Lookup (directory)** | A mapping table says where each key lives | Very flexible; move one tenant easily | Extra lookup step; the map must stay consistent |

The Elastic Database client library has only **list** and **range** shard maps. For hash, the app computes a bucket and maps buckets with a list or range map.

**Example:** A photo app stores users by `UserID % 8` (hash). Load is even. But "all users who joined between ID 1000 and 2000" must ask all 8 shards.

## Multi-tenant patterns (SaaS)
| Pattern | Isolation | Cost per tenant | Scale |
|---------|-----------|-----------------|-------|
| Standalone app + single DB per tenant | Highest | Highest | 1–100s |
| **Database per tenant** | High | Low with **elastic pools** | Up to 100,000s |
| **Sharded multi-tenant DBs** | Low | **Lowest** | Up to millions |
| Hybrid | Mixed | Mixed | Very high |

- Database per tenant = **list** mapping (one key → one database). Easy per-tenant restore.
- Multi-tenant shards = **range** mapping. Needs a tenant ID in every table and **row-level security**.
- You **can't restore one tenant** from a multi-tenant database. Move it out with split-merge or your own ETL.

## Elastic Database tools (SQL Database only)
| Tool | Purpose | Status |
|------|---------|--------|
| **Elastic Database client library** | Builds the shard map; **data-dependent routing** (`OpenConnectionForKey`) | Supported |
| **Shard map manager** | Global shard map database + a local map in each shard | Part of the library |
| **Split-merge tool** | Moves **shardlets** (all data for one key) between shards | Legacy (built on retired Cloud Services) |
| **Elastic query** | Read-only T-SQL across databases via external tables | Preview. Horizontal (`SHARD_MAP_MANAGER`) mode **ends 31 Mar 2027** |
| **Elastic jobs** | Run T-SQL on many databases on a schedule | GA |
| **Elastic transactions** | Distributed transactions across SQL databases | Supported |

- Shard map methods change **metadata only**. They never move data.
- Elastic query: read-only, **SQL authentication only**, one offline shard fails the whole query.
- Elastic jobs: target a **server** or **pool** and new shards are picked up automatically. Use a **user-assigned managed identity** (system-assigned is not supported). Job database S1 or higher, **not Hyperscale**. Scripts must be safe to re-run.
- On **Managed Instance**, use cross-database queries or linked servers instead of these tools.

## Remember for the exam
- Dates → **RANGE RIGHT**. N boundaries = N + 1 partitions.
- "Remove a month of data instantly" → **SWITCH out** or `TRUNCATE ... WITH (PARTITIONS)`, not DELETE.
- "Old partitions on cheaper storage" → filegroups on **VM or MI**. Not possible in SQL Database.
- SWITCH fails → check target empty, same filegroup, aligned indexes, **same compression**.
- Cold columnstore → **COLUMNSTORE_ARCHIVE**. Estimate first → `sp_estimate_data_compression_savings`.
- "50 TB, minimal app changes" → **Hyperscale**, not sharding.
- "Thousands of tenants, isolation, per-tenant restore, low cost" → **database per tenant in elastic pools**.
- "Millions of tiny tenants, lowest cost" → **sharded multi-tenant + RLS**.
- "Route a connection to the right shard" → **Elastic Database client library**.
- "Same schema change on all shards, including future ones" → **elastic job** targeting the server or pool.

## Quick check
1. A monthly partition switch fails. The archive table is empty and on the same filegroup. The source partition uses PAGE compression; the archive table uses none. What is wrong?
   <details><summary>Answer</summary>Compression must match between source and target. Rebuild the archive table with PAGE.</details>
2. Which shard strategy avoids hot spots but makes range queries expensive?
   <details><summary>Answer</summary>Hash sharding. It spreads rows evenly, so a range is scattered over all shards.</details>
3. You want to keep old order partitions on cheaper disks. Can you do this in Azure SQL Database?
   <details><summary>Answer</summary>No. SQL Database has only the PRIMARY filegroup. Use a VM or Managed Instance.</details>
