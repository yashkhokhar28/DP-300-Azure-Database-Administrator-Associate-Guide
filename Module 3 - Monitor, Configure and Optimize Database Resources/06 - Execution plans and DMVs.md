# Execution plans and DMVs

> **Exam skills:** Review execution plans, Identify performance issues using dynamic management views (DMVs)
> **Reading time:** ~9 min

## The big idea
An **execution plan** is the step-by-step recipe SQL Server follows to run a query. Reading it shows *why* a query is slow. **DMVs** (dynamic management views) are built-in system views that show what is happening inside SQL Server right now: running queries, locks, waits, index use.

Think of a satnav. The plan is the route it chose. If it sends you through every small street (a table scan) instead of the motorway (an index seek), the trip is slow. DMVs are the live traffic map.

## How SQL Server picks a plan
SQL Server uses a **cost-based optimizer**. It estimates the "cost" of possible plans and picks the cheapest one it finds. It does not try every plan; it uses shortcuts.

Steps for each query:
1. **Parse** – check the syntax.
2. **Bind (algebrizer)** – check that tables and columns exist; check data types.
3. **Plan cache check** – reuse a plan if one is already in the **plan cache** (memory that stores plans).
4. **Optimize** – if no plan is cached, build one using **statistics**, **indexes** and the cost of joins, sorts and filters.
5. **Execute.**

The optimizer depends on **statistics**. If they are old, the row guesses are wrong and the plan is bad.

**Example:** The optimizer thinks `WHERE City = 'Leeds'` returns 100 rows, so it uses an index seek plus lookups. Really it returns 2 million rows. A scan would have been much faster. Updating statistics fixes the guess.

### Memory grants
Each plan reserves memory for sorts and hashes (a **memory grant**).
- Grant **too big** → memory wasted; other queries wait with `RESOURCE_SEMAPHORE`.
- Grant **too small** → data **spills to tempdb** (disk), which is slow. The plan shows a spill warning.

## Estimated vs actual plan
| | Estimated plan | Actual plan |
|--|----------------|-------------|
| Runs the query? | **No** | **Yes** |
| Shows | Estimated rows and costs | Real rows, real memory, warnings, plus estimates |
| Use when | Writing or checking a query safely | Troubleshooting a slow query |
| SSMS | **Ctrl + L** | **Ctrl + M**, then run |
| T-SQL | `SET SHOWPLAN_ALL ON` / `SET SHOWPLAN_XML ON` | `SET STATISTICS PROFILE ON` / `SET STATISTICS XML ON` |

- The **shape** of the plan is almost always the same. Only the actual plan has runtime numbers.
- **Live Query Statistics** shows the plan in real time while a long query runs.

**Big warning sign:** **estimated rows very different from actual rows**. This usually means old statistics or parameter sniffing.

## Reading a plan
- Read **right to left, top to bottom**. Data flows from the right toward the `SELECT` on the left.
- Each icon is an **operator** (Scan, Seek, Join, Sort...).
- **Cost %** shows each step's share of the total.
- **Thicker arrows** = more rows.
- Hover for a tooltip; right-click → **Properties** for full details.

### Operators to know
| Operator | Meaning | Good or bad? |
|----------|---------|--------------|
| **Index Seek** | Jumps straight to the needed rows | Usually good |
| **Index Scan / Clustered Index Scan / Table Scan** | Reads the whole index or table | Bad on big tables when you need few rows |
| **Key Lookup** (or RID Lookup on a heap) | Goes back to the table for columns missing from the index | Costly when repeated many times; fix with a **covering index** |
| **Nested Loops / Hash Match / Merge Join** | Join types: small inputs / large unsorted (needs memory) / large sorted | Depends on row counts |
| **Sort** | Orders rows; needs memory | Watch for spills |
| **Yellow warning triangle** | Spill, implicit conversion, missing statistics, missing index | Investigate |

**Example:** A shop's "orders by customer" query takes 8 seconds. The plan shows a Clustered Index Scan on Orders with 90% of the cost and a green "missing index" hint on `CustomerID`. After adding that index, the plan shows an Index Seek and the query takes 50 ms.

## Lightweight profiling and the last actual plan
**Lightweight query profiling** costs about **2%** and is **on by default** in SQL Server 2019+ and Azure SQL. Ways to turn it on in older versions: trace flag 7412, the hint `USE HINT('QUERY_PLAN_PROFILE')` for one query, or the scoped config `LIGHTWEIGHT_QUERY_PROFILING`.

To see the **last actual plan** of a cached query without running it again:
```sql
ALTER DATABASE SCOPED CONFIGURATION SET LAST_QUERY_PLAN_STATS = ON;
SELECT st.text, qps.query_plan
FROM sys.dm_exec_cached_plans AS cp
CROSS APPLY sys.dm_exec_sql_text(cp.plan_handle) AS st
CROSS APPLY sys.dm_exec_query_plan_stats(cp.plan_handle) AS qps;
```
`sys.dm_exec_query_plan_stats` works in SQL Server 2019+, Azure SQL Database and MI. It is **opt-in**: turn on `LAST_QUERY_PLAN_STATS` per database, or trace flag 2451 for the whole server.

## Dynamic management views (DMVs)
- DMVs and DMFs (functions) all start with **`sys.dm_`**.
- They show live internal data: running queries, sessions, waits, locks, cached plans, index use, resource use.
- Most data resets when the server restarts. For long-term history, use **Query Store**.

### Permissions
| Scope | Permission |
|-------|------------|
| Server-scoped DMVs | `VIEW SERVER STATE` |
| Database-scoped DMVs | `VIEW DATABASE STATE` |

SQL Server 2022+ adds finer permissions such as `VIEW SERVER PERFORMANCE STATE` and `VIEW DATABASE PERFORMANCE STATE`. In Azure SQL Database, the **`##MS_ServerStateReader##`** server role gives DMV access.

### DMVs to know
| Area | DMV | Shows |
|------|-----|-------|
| Running work | `sys.dm_exec_requests` | Running queries, wait type, blocking session |
| | `sys.dm_exec_sessions` | Connected sessions |
| Query cost | `sys.dm_exec_query_stats` | CPU, reads, duration of cached queries |
| | `sys.dm_exec_procedure_stats` | Same, per stored procedure |
| Plans and text | `sys.dm_exec_cached_plans`, `sys.dm_exec_sql_text`, `sys.dm_exec_query_plan` | Cached plans and their query text |
| Waits | `sys.dm_os_wait_stats` | Waits since start (instance) |
| | `sys.dm_db_wait_stats` | Waits for one Azure SQL database |
| | `sys.dm_exec_session_wait_stats` | Waits per session |
| | `sys.dm_os_waiting_tasks` | Tasks waiting right now |
| Locks | `sys.dm_tran_locks` | Current locks |
| | `sys.dm_tran_active_transactions` | Open transactions |
| Indexes | `sys.dm_db_index_usage_stats` | Seeks, scans, lookups, updates per index |
| | `sys.dm_db_index_physical_stats` | Fragmentation |
| | `sys.dm_db_missing_index_details` | Indexes SQL Server wishes it had |
| Resources (Azure) | `sys.dm_db_resource_stats` | CPU, I/O, log, memory for an Azure SQL database |
| | `sys.server_resource_stats` | Same for Managed Instance |

- `sys.dm_db_resource_stats` exists **only in Azure SQL**.
- In Azure SQL Database, use database-scoped `sys.dm_db_wait_stats` for wait statistics.

### Find the top CPU queries
```sql
SELECT TOP 5 qs.total_worker_time / qs.execution_count AS avg_cpu,
       qs.execution_count, st.text
FROM sys.dm_exec_query_stats AS qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) AS st
ORDER BY qs.total_worker_time DESC;
```

**Example:** At 11 a.m. a bank's app hangs. The DBA runs `sys.dm_exec_requests` and sees 40 sessions waiting with `LCK_M_X`, all blocked by session 77. `sys.dm_exec_sessions` shows session 77 belongs to a user who left a transaction open. (Lesson 8 shows how to fix blocking.)

## Remember for the exam
- **Estimated plan** = does not run the query (Ctrl+L). **Actual plan** = runs it and shows real rows (Ctrl+M).
- Read plans **right to left**. Scans on big tables, Key Lookups, spills and warnings are red flags.
- Big gap between estimated and actual rows → **statistics** or **parameter sniffing**.
- **Key Lookup** → add a **covering index** (INCLUDE columns).
- Last actual plan → `sys.dm_exec_query_plan_stats` with **LAST_QUERY_PLAN_STATS = ON**.
- DMVs start with `sys.dm_`; need **VIEW SERVER STATE** or **VIEW DATABASE STATE**.
- Live running queries → `sys.dm_exec_requests`. Locks → `sys.dm_tran_locks`. Missing indexes → `sys.dm_db_missing_index_details`.
- Azure SQL DB resource use → `sys.dm_db_resource_stats`; MI → `sys.server_resource_stats`.

## Quick check
1. You must see the plan of a heavy query without running it on production. What do you use?
   <details><summary>Answer</summary>The estimated execution plan (Ctrl+L or SET SHOWPLAN_XML ON). It does not run the query.</details>
2. A plan shows an Index Seek on `ProductID` followed by thousands of Key Lookups to get `Price`. What fix?
   <details><summary>Answer</summary>Create a covering nonclustered index on ProductID with INCLUDE (Price) so no lookup is needed.</details>
3. Which permission does a user need to query `sys.dm_db_wait_stats` in an Azure SQL database?
   <details><summary>Answer</summary>VIEW DATABASE STATE, because it is a database-scoped DMV.</details>
