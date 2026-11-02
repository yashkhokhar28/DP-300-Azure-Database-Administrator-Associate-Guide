# Query Store

> **Exam skills:** Configure Query Store, Monitor by using Query Store
> **Reading time:** ~8 min

## The big idea
Without Query Store, SQL Server keeps only the **current** plan of each query in memory, and it is lost on restart. **Query Store** is a recorder inside the database. It saves query text, every plan used, run-time numbers and waits, and keeps them **after restarts**. You can see when a query got slower and **force** the old, faster plan.

Think of a flight's black box. After a problem, you replay what happened and see exactly when things went wrong.

## What Query Store saves
| Store | Contents |
|-------|----------|
| **Plan store** | Query text and execution plans, including forced plans |
| **Runtime stats store** | CPU, duration, reads, row counts per plan per time interval |
| **Wait stats store** | Waits per query (I/O, locks, memory...) |

- Data is collected when a query first runs, recompiles, and on each run.
- It is written to disk **asynchronously** (in the background), so the cost is small.

## Turning it on
- **On by default** in Azure SQL Database, Azure SQL Managed Instance and **new** databases in SQL Server 2022+.
- In Azure SQL Database single and pooled databases it **cannot be turned off**.
- In SQL Server 2016–2019 turn it on yourself:

```sql
ALTER DATABASE SalesDb SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE);
```

## Operation modes
| Mode | Meaning |
|------|---------|
| **READ_WRITE** | Normal: collects new data |
| **READ_ONLY** | Stops collecting; you can still read old data |
| **OFF** | Disabled |

**Why does Query Store switch to READ_ONLY by itself?**
- It reached **MAX_STORAGE_SIZE_MB** (most common).
- Fix: raise the max size, turn on size-based cleanup, lower the stale query threshold, or capture fewer queries. Then set it back to READ_WRITE.
- Check the current and desired mode in `sys.database_query_store_options` (`actual_state_desc`, `desired_state_desc`, `readonly_reason`).

## Key settings (SQL Server 2019+ / Azure SQL DB defaults)
| Setting | Default | Meaning |
|---------|---------|---------|
| `OPERATION_MODE` | READ_WRITE | Collect or not |
| `QUERY_CAPTURE_MODE` | **AUTO** (was ALL in 2016/2017) | AUTO skips tiny, rare queries; ALL catches everything; NONE catches no new queries; CUSTOM lets you set rules |
| `MAX_STORAGE_SIZE_MB` | **1000** (was 100) | Size cap; when full → READ_ONLY |
| `SIZE_BASED_CLEANUP_MODE` | AUTO | Deletes oldest data as it nears the cap |
| `STALE_QUERY_THRESHOLD_DAYS` | **30** | How long to keep data |
| `DATA_FLUSH_INTERVAL_SECONDS` | **900** (15 min) | How often memory data is written to disk |
| `INTERVAL_LENGTH_MINUTES` | **60** | Size of each statistics time bucket |

```sql
ALTER DATABASE SalesDb SET QUERY_STORE (
    MAX_STORAGE_SIZE_MB = 2048,
    QUERY_CAPTURE_MODE = AUTO,
    SIZE_BASED_CLEANUP_MODE = AUTO,
    CLEANUP_POLICY = (STALE_QUERY_THRESHOLD_DAYS = 30));
```

**Exam trap:** the **data flush interval** controls how often data is written to disk. The **statistics collection interval** (`INTERVAL_LENGTH_MINUTES`) only controls how runtime numbers are grouped. Questions about write pressure point to the **data flush interval**. Questions about Query Store becoming read-only because it is full point to **max size and cleanup**.

**Example:** A busy shop database's Query Store keeps going READ_ONLY every two weeks. `readonly_reason` shows it is full. The DBA raises MAX_STORAGE_SIZE_MB to 2048 and sets SIZE_BASED_CLEANUP_MODE = AUTO. It stays in READ_WRITE.

**Query Store for readable secondaries** (preview): always on in Azure SQL Database and MI (Always-up-to-date policy); opt-in in SQL Server 2025 with `ALTER DATABASE ... FOR SECONDARY SET QUERY_STORE = ON`.

## Built-in reports (SSMS → database → Query Store)
| Report | Use it to |
|--------|-----------|
| **Regressed Queries** | Find queries that got slower; compare old and new plans; **force** the old plan |
| **Overall Resource Consumption** | See total CPU, duration, reads over time and drill into the cause |
| **Top Resource Consuming Queries** | Find the top 25 most expensive queries |
| **Queries With Forced Plans** | See and **unforce** forced plans |
| **Queries With High Variation** | Find queries that are sometimes fast, sometimes slow (often parameter sniffing) |
| **Query Wait Statistics** | See wait categories and the queries causing them |
| **Tracked Queries** | Follow one query's history by its ID |

## Forcing and unforcing a plan
```sql
EXEC sp_query_store_force_plan   @query_id = 73, @plan_id = 79;
EXEC sp_query_store_unforce_plan @query_id = 73, @plan_id = 79;
```

**Example:** After a statistics update, the monthly invoice procedure takes 12 seconds instead of 1. In **Regressed Queries**, the DBA sees plan 79 (old, 1 second) and plan 85 (new, 12 seconds). They force plan 79. The procedure is fast again while they look for the real cause.

Automatic tuning (lesson 5) can do the same thing for you with **FORCE_LAST_GOOD_PLAN**.

## Query Store hints
Sometimes you cannot change the query text, for example in a vendor app. **Query Store hints** attach a hint to a query **without changing its code**. Available in Azure SQL Database, MI and SQL Server 2022+.

```sql
-- 1. find the query_id
SELECT q.query_id, qt.query_sql_text
FROM sys.query_store_query_text AS qt
JOIN sys.query_store_query AS q ON qt.query_text_id = q.query_text_id
WHERE qt.query_sql_text LIKE N'%ORDER BY CustomerName DESC%';
-- 2. add the hint
EXEC sys.sp_query_store_set_hints @query_id = 42, @query_hints = N'OPTION(RECOMPILE, MAXDOP 1)';
```
- See hints in `sys.query_store_query_hints`. Remove with `sys.sp_query_store_clear_hints @query_id = 42`.
- Hints survive restarts and failovers.
- They **override** hints written in the query and **plan guides**.

**Example:** A vendor's billing app runs a hard-coded query that suffers from parameter sniffing. The company may not edit the app. The DBA adds a Query Store hint `OPTION(RECOMPILE)` to that query_id. The problem goes away and no code changed.

## Catalog views to know
| View | Shows |
|------|-------|
| `sys.database_query_store_options` | Current settings and state |
| `sys.query_store_query` / `sys.query_store_query_text` | Queries and their text |
| `sys.query_store_plan` | Plans, including `is_forced_plan` |
| `sys.query_store_runtime_stats` | Runtime numbers per interval |
| `sys.query_store_wait_stats` | Waits per plan |

## Remember for the exam
- Query Store is **on by default** in Azure SQL DB, MI and new SQL Server 2022+ databases. Turn it on with `SET QUERY_STORE = ON`.
- Full → goes **READ_ONLY**. Fix with bigger **MAX_STORAGE_SIZE_MB** and **cleanup**.
- Write-pressure question → **DATA_FLUSH_INTERVAL_SECONDS** (default 900), not the statistics interval.
- Capture mode **AUTO** skips trivial queries; **ALL** catches everything.
- Find slower queries → **Regressed Queries** report. Fix → **force the plan** with `sp_query_store_force_plan`.
- Sometimes fast, sometimes slow → **Queries With High Variation**.
- Cannot change app code → **Query Store hints** (`sp_query_store_set_hints`).
- Automatic plan correction relies on Query Store.

## Quick check
1. Query Store in your database shows `actual_state_desc = READ_ONLY` and the reason is size. What do you do?
   <details><summary>Answer</summary>Increase MAX_STORAGE_SIZE_MB and/or enable size-based cleanup and a shorter stale threshold, then set OPERATION_MODE back to READ_WRITE.</details>
2. A procedure became slow after a deployment. You want the old plan back right away. What do you use?
   <details><summary>Answer</summary>The Regressed Queries report to find the old plan, then force it (in SSMS or with sp_query_store_force_plan).</details>
3. A third-party app sends a query you cannot edit, and it needs MAXDOP 1. How do you apply it?
   <details><summary>Answer</summary>A Query Store hint: sys.sp_query_store_set_hints with N'OPTION(MAXDOP 1)' for that query_id.</details>
