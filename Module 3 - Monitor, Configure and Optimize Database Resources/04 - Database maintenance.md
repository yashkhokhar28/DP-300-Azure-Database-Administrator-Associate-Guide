# Database maintenance

> **Exam skills:** Implement index maintenance tasks, Implement statistics maintenance tasks, Implement database integrity checks, Implement database-scoped configuration
> **Reading time:** ~10 min

## The big idea
Data changes every day. Indexes get messy, statistics get old, and very rarely a page gets damaged. Regular maintenance keeps the database fast and healthy. Database-scoped configuration lets you tune settings for one database without touching the others on the same server.

Think of a library. Books get put back in the wrong place (fragmented indexes). The catalogue says there are 10 copies when there are 50 (old statistics). Sometimes a page is torn (corruption). A librarian tidies shelves, updates the catalogue and checks for damage.

## Index maintenance: reorganize or rebuild
**Fragmentation** means index pages are no longer in order, or are half empty. SQL Server then reads more pages than needed.

Find fragmentation with:
- `sys.dm_db_index_physical_stats` – normal (rowstore) indexes.
- `sys.dm_db_column_store_row_group_physical_stats` – columnstore indexes.

| Fragmentation (rule of thumb) | Action | Notes |
|-------------------------------|--------|-------|
| 5% – 30% | **REORGANIZE** | Always **online**. Light. Does **not** update statistics. |
| Over 30% | **REBUILD** | Creates the index again. Can be **online** or offline. **Updates statistics** (full scan). Needs more time and space. |

```sql
ALTER INDEX IX_Orders_Date ON dbo.Orders REORGANIZE;
ALTER INDEX IX_Orders_Date ON dbo.Orders REBUILD WITH (ONLINE = ON, RESUMABLE = ON);
```

- The 5% / 30% numbers come from the Learn module. Current Microsoft docs warn **not to rely on fixed numbers alone**. Measure the benefit. Often **updating statistics** gives the same speed-up for much less work.
- In **Azure SQL Database and MI**, prefer **REORGANIZE** or **online + resumable** rebuilds. Resumable operations survive a failover.

### Resumable index operations
- You can **PAUSE** and **RESUME** a large index build or rebuild.
- Resumable **rebuild**: SQL Server 2017+. Resumable **create**: SQL Server 2019+. Both in Azure SQL Database and MI.
- Works only with **ONLINE = ON**.
- Commands: `ALTER INDEX ... PAUSE`, `... RESUME`, `... ABORT`. Limit run time with `MAX_DURATION = <minutes>`.
- Check progress in `sys.index_resumable_operations`.
- Database-scoped settings **ELEVATE_ONLINE** and **ELEVATE_RESUMABLE** can make online/resumable the default.

**Example:** A rebuild of a 500 GB index takes 6 hours, but the maintenance window is only 2 hours each night. The DBA starts it with `RESUMABLE = ON, MAX_DURATION = 120`. It pauses after 2 hours and resumes the next night.

Note: **Automatic index compaction** (`SET AUTOMATIC_INDEX_COMPACTION = ON`, off by default) in Azure SQL Database and MI keeps recently changed pages full. It does **not** reduce fragmentation or update statistics.

## Statistics maintenance
**Statistics** describe how values are spread in a column (for example, how many orders per city). The **query optimizer** (the part of SQL Server that picks a plan) uses them to guess row counts (**cardinality**). Old statistics → bad guesses → slow plans.

- `AUTO_CREATE_STATISTICS` and `AUTO_UPDATE_STATISTICS` are **on by default**. SQL Server creates and refreshes statistics on its own.
- You still update them by hand after big data loads or when plans go wrong.

```sql
UPDATE STATISTICS dbo.Orders;                     -- one table
UPDATE STATISTICS dbo.Orders WITH FULLSCAN;       -- read every row
EXEC sp_updatestats;                              -- whole database
CREATE STATISTICS st_City ON dbo.Customers (City);
```

Check when statistics were last updated:
```sql
SELECT s.name, sp.last_updated, sp.rows, sp.rows_sampled, sp.modification_counter
FROM sys.stats AS s
CROSS APPLY sys.dm_db_stats_properties(s.object_id, s.stats_id) AS sp;
```

**Create statistics by hand when:**
- A query filters on several columns that no index covers.
- The query reads a small subset of data, or the plan warns about missing statistics.
- The Database Engine Tuning Advisor suggests it.

**Example:** Every night a school loads 2 million new exam results. Next morning a report is slow because the optimizer still thinks the table is small. Adding `UPDATE STATISTICS` to the end of the load job fixes it.

## Database integrity checks (DBCC CHECKDB)
**DBCC CHECKDB** checks a whole database for **corruption** (damaged pages or broken links between tables and indexes).

```sql
DBCC CHECKDB ('SalesDb') WITH NO_INFOMSGS;
```

- **SQL Server on a VM:** this is **your job**. Schedule it with a **SQL Server Agent job** or a **maintenance plan** ("Check Database Integrity" task).
- **Azure SQL Database and Managed Instance:** Azure runs **integrity checks automatically**, together with lost-write detection and automatic page repair. You can still run DBCC CHECKDB yourself.
- CHECKDB uses a lot of CPU and I/O. Run it **off-hours** (for example at weekends).
- Keep backups **longer than the gap between checks**. If corruption is found, you need a backup from before it started.
- A backup is only proven good after a **test restore + DBCC CHECKDB**. `RESTORE VERIFYONLY` alone does not prove the data is consistent.
- Do **not** shrink databases as a routine task; shrinking causes fragmentation.

**Example:** A hospital runs SQL Server on a VM. A weekly Agent job runs CHECKDB on Sunday at 2 a.m. and keeps 2 weeks of backups, so a clean backup always exists if corruption shows up.

## Automating maintenance
Schedule maintenance with **SQL Server Agent** on a VM or Managed Instance. Azure SQL Database has **no Agent**, so use **elastic jobs** or **Azure Automation runbooks** (see Module 4). In Azure SQL Database and MI, backups and integrity checks are done by the platform, but **index and statistics maintenance is still your job**.

## Database-scoped configuration
Older versions had many settings only at the **instance** level (whole server). Now many can be set **per database**. This helps when one server hosts many different apps.

Two commands set database-level options:
- `ALTER DATABASE` – recovery model, automatic tuning, auto stats, Query Store, snapshot isolation (SI and RCSI), compatibility level.
- `ALTER DATABASE SCOPED CONFIGURATION` – query-processing options.

| Option (scoped configuration) | What it does |
|-------------------------------|--------------|
| `MAXDOP` | Max cores per query in this database, whatever the server setting |
| `LEGACY_CARDINALITY_ESTIMATION` | Uses the old row-estimate model; helps some old apps |
| `PARAMETER_SNIFFING` | Turn parameter sniffing on or off |
| `LAST_QUERY_PLAN_STATS` | Keeps the last actual plan for each query |
| `OPTIMIZE_FOR_AD_HOC_WORKLOADS` | Stores only a small plan stub for single-use queries |
| `LIGHTWEIGHT_QUERY_PROFILING` | Low-cost collection of runtime plan data |
| `ELEVATE_ONLINE` / `ELEVATE_RESUMABLE` | Make index operations online / resumable by default |
| `DOP_FEEDBACK` | Turn DOP feedback on (lesson 5) |

```sql
ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = 4;
ALTER DATABASE SCOPED CONFIGURATION SET LEGACY_CARDINALITY_ESTIMATION = ON;
ALTER DATABASE SCOPED CONFIGURATION CLEAR PROCEDURE_CACHE;
```

### Compatibility level
- Each database has a **compatibility level**, for example 120, 140, 150, 160, 170. It decides how the optimizer behaves.
- Newer levels turn on **Intelligent Query Processing** features (lesson 5).
- Default for **new** databases: **170** in Azure SQL Database and SQL Server 2025. Managed Instance: **160** with the SQL Server 2022 update policy, **170** with the SQL Server 2025 / Always-up-to-date policy.
- Restored or migrated databases **keep their old level**. Test, then raise it.

```sql
ALTER DATABASE SalesDb SET COMPATIBILITY_LEVEL = 160;
```

**Example:** A hosting company runs 20 customer databases on one server. One old app gets slower after an upgrade. The DBA turns on `LEGACY_CARDINALITY_ESTIMATION` **for that database only**. The other 19 keep the new behaviour.

## Remember for the exam
- **REORGANIZE**: always online, light, does **not** update stats. **REBUILD**: updates stats, online or offline.
- Rule of thumb: 5–30% reorganize, over 30% rebuild (but measure; stats updates are often enough).
- **Resumable** needs **ONLINE = ON**; use PAUSE / RESUME / ABORT and MAX_DURATION.
- Fragmentation DMV: `sys.dm_db_index_physical_stats`. Stats DMF: `sys.dm_db_stats_properties`.
- **DBCC CHECKDB** = corruption check. On a VM you schedule it; Azure SQL DB and MI do it automatically.
- Azure SQL Database has **no SQL Agent** → elastic jobs or Automation runbooks.
- `ALTER DATABASE SCOPED CONFIGURATION` = per-database MAXDOP, legacy CE, parameter sniffing, last query plan stats.
- Raise the **compatibility level** to get newer IQP features; migrated databases keep their old level.

## Quick check
1. An index is 12% fragmented and must stay available to users. Which action?
   <details><summary>Answer</summary>ALTER INDEX ... REORGANIZE. It is always online and suits light to medium fragmentation.</details>
2. You need to check for corruption in SQL Server on an Azure VM every week. What do you do?
   <details><summary>Answer</summary>Schedule DBCC CHECKDB with a SQL Server Agent job or a maintenance plan. On a VM the platform does not do it for you.</details>
3. One database on a shared server needs the old cardinality estimator, but the others must not change. How?
   <details><summary>Answer</summary>ALTER DATABASE SCOPED CONFIGURATION SET LEGACY_CARDINALITY_ESTIMATION = ON in that database only.</details>
