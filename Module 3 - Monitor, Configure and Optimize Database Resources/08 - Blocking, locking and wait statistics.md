# Blocking, locking and wait statistics

> **Exam skills:** Identify and resolve session blocking, Identify performance issues using dynamic management views (DMVs)
> **Reading time:** ~11 min

## The big idea
When many people use the same data, SQL Server uses **locks** so they do not damage each other's changes. If one session holds a lock for a long time, others must wait. This is **blocking**. **Wait statistics** record *why* queries wait (locks, disk, memory, CPU). Together they tell you where time is lost.

Think of a single shop changing room. One person goes in and locks the door (lock). The queue outside waits (blocking). If someone falls asleep inside (an open transaction), the whole queue is stuck. A note on the wall saying "people waited 2 hours for the changing room today" is a wait statistic.

## Locking
- Locks protect **ACID** rules (Atomicity, Consistency, Isolation, Durability: transactions are all-or-nothing, correct, separate and permanent).
- SQL Server takes locks **automatically** when you read or change data.
- Lock sizes: **row** (best sharing), **page**, **table** (least sharing).
- **Lock escalation:** if one statement takes about **5,000** row locks, SQL Server may swap them for one table lock to save memory. This can block many users.

## Blocking and deadlocks
- **Blocking:** Session A holds a lock; Session B wants the same data and must wait. Short blocking is normal.
- Blocking becomes a problem when:
  1. A lock is held **too long** (long transaction).
  2. A lock is **never released**, for example a transaction with no `COMMIT` or `ROLLBACK`.
- **Deadlock:** A waits for B, and B waits for A. Neither can move. SQL Server detects this and kills one session (the **deadlock victim**, error 1205). The other continues.

```sql
BEGIN TRANSACTION;
UPDATE dbo.Stock SET Qty = Qty - 1 WHERE ProductID = 5;
COMMIT TRANSACTION;   -- forget this line and row 5 stays locked
```

## Finding blocking
```sql
SELECT session_id, blocking_session_id, wait_type, wait_time, status
FROM sys.dm_exec_requests
WHERE blocking_session_id <> 0;
```
- `sys.dm_exec_requests` – who is blocked and by whom (`blocking_session_id`).
- `sys.dm_tran_locks` – which locks are held or requested.
- `sys.dm_os_waiting_tasks` – tasks waiting right now and on what.
- `sys.dm_tran_active_transactions` / `sys.dm_tran_session_transactions` – open transactions and how long they have been open.
- **Extended Events** (the `blocked_process_report` and deadlock events) or **database watcher** for ongoing monitoring.

**Find the head blocker:** follow `blocking_session_id` until you reach a session that is not blocked itself.

## Resolving blocking
- **Short term:** find the head blocker. If it is safe, ask the user to commit, or end it with `KILL <session_id>` (its work is rolled back).
- **Long term:**
  - Keep transactions **short**. Never wait for user input inside a transaction.
  - Always **COMMIT or ROLLBACK**; handle errors so transactions do not stay open.
  - Add **indexes** so updates touch fewer rows (a scan locks many more rows than a seek).
  - Update in **smaller batches** to avoid lock escalation.
  - Use **row versioning** isolation (RCSI or snapshot) so readers do not block writers.
  - Access tables in the **same order** in all code to avoid deadlocks.

**Example:** Every lunchtime a warehouse app freezes. `sys.dm_exec_requests` shows 30 sessions blocked by session 61. Session 61 is a report that runs `UPDATE` on Stock without a `WHERE` index, scanning and locking the whole table. Adding an index on the filter column and running the update in batches of 1,000 rows ends the freezes.

## Isolation levels
An **isolation level** decides how much one transaction is protected from others. More protection = more blocking.

| Isolation level | Dirty reads? | Holds read locks until end? | Uses row versions? |
|-----------------|--------------|-----------------------------|--------------------|
| Read uncommitted | Yes | No | No |
| **Read committed** (SQL Server default) | No | No | No (unless RCSI is on) |
| Repeatable read | No | Yes, for rows read | No |
| Serializable | No | Yes, for the whole range | No |
| **Read committed snapshot (RCSI)** | No | No | Yes |
| **Snapshot** | No | No (whole transaction sees one version; update conflicts possible) | Yes |

A **dirty read** = reading data another transaction has changed but not yet committed.

```sql
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
ALTER DATABASE SalesDb SET READ_COMMITTED_SNAPSHOT ON;
```

### Row versioning
- With RCSI or snapshot, readers read an **older committed version** of the row instead of waiting for a lock. **Readers do not block writers, and writers do not block readers.**
- Old row versions are kept in **tempdb**. With **Accelerated Database Recovery (ADR)**, always on in Azure SQL Database and MI, they go to the **persistent version store (PVS)** inside the user database.
- **RCSI is ON by default in Azure SQL Database** (not in SQL Server or MI).

### Optimized locking
- Always on in **Azure SQL Database**, and in MI with the Always-up-to-date or SQL Server 2025 update policy. Opt-in in SQL Server 2025 (needs ADR).
- **TID locking:** holds one lock on the transaction ID instead of many row and page locks.
- **Lock after qualification (LAQ):** needs RCSI; checks the filter on the latest committed version before taking a lock.
- Result: less blocking, less lock memory, almost no lock escalation.

**Example:** A school app on SQL Server has teachers' reports blocked by mark updates all day. The DBA turns on **READ_COMMITTED_SNAPSHOT**. Reports now read the last committed marks without waiting, and updates are no longer blocked by reports.

## Wait statistics
Every time a task must wait, SQL Server records the **wait type** and time.
- **Resource waits** – for a lock, memory, disk, etc.
- **Queue waits** – idle workers waiting for work (usually harmless).
- **External waits** – something outside SQL Server, such as a linked server.

Where to see them:
- `sys.dm_os_wait_stats` – instance totals since start.
- `sys.dm_db_wait_stats` – one Azure SQL database.
- `sys.dm_exec_session_wait_stats` – per session.
- **Query Store** wait stats – per query plan.

```sql
SELECT TOP 10 wait_type, wait_time_ms, waiting_tasks_count
FROM sys.dm_os_wait_stats
ORDER BY wait_time_ms DESC;
-- reset the counters (SQL Server):
DBCC SQLPERF('sys.dm_os_wait_stats', CLEAR);
```

### Common wait types
| Wait type | Meaning | Likely fix |
|-----------|---------|-----------|
| **LCK_M_X**, **LCK_M_S** (LCK_M_*) | Waiting for a lock (blocking) | Shorter transactions, indexes, RCSI |
| **PAGEIOLATCH_SH** | Waiting to read a data page from **disk** | Faster storage, or indexes to avoid scans, more memory |
| **PAGELATCH_UP / _EX** | Contention on in-memory pages (often tempdb allocation pages, or last-page inserts). No disk I/O. | More equal tempdb files; `OPTIMIZE_FOR_SEQUENTIAL_KEY` |
| **RESOURCE_SEMAPHORE** | Waiting for a **memory grant** | Tune big queries, fix bad estimates, add memory |
| **SOS_SCHEDULER_YIELD** | High **CPU** pressure | Tune CPU-heavy queries or scale up |
| **CXPACKET** | Parallel threads waiting for each other | Lower MAXDOP, raise cost threshold for parallelism |
| **WRITELOG** | Waiting for log writes | Faster log disk |
| **ASYNC_NETWORK_IO** | Client is slow to read results | Fix the app or network |
| **THREADPOOL** | No free worker threads | Fewer concurrent requests, scale up |

**Example:** Users say the app is slow. Top wait is `PAGEIOLATCH_SH`. The DBA checks the plans of the top queries and finds big table scans. Adding two indexes cuts the disk reads, and the wait drops.

## Running or waiting? A simple troubleshooting path
1. Check **CPU** (portal or `sys.dm_db_resource_stats`).
2. **High CPU (running problem):** find top CPU queries with Query Store, `sys.dm_exec_query_stats`, `sys.dm_exec_procedure_stats`, `sys.dm_exec_requests`. Tune them or scale up.
3. **Normal CPU (waiting problem):** check wait stats and `sys.dm_os_waiting_tasks`. Then fix locks, I/O, memory, or scale.

### Azure-specific waits
| Wait / error | Meaning |
|--------------|---------|
| `LOG_RATE_GOVERNOR` | Azure SQL database hit its **log write limit** |
| `POOL_LOG_RATE_GOVERNOR` | Elastic pool hit its log limit |
| `INSTANCE_LOG_RATE_GOVERNOR` | Managed Instance log limit |
| `HADR_SYNC_COMMIT` | Business Critical: waiting for a replica to confirm a commit |
| `HADR_THROTTLE_LOG_RATE_*` | Log rate throttled because a replica or geo-secondary is lagging |
| `RBIO_*` | Hyperscale page server reads |
| Error **10928** | Azure SQL Database hit its **worker limit** (the message says "request limit") |

Worker and session limits depend on the **service tier and compute size**. On MI, too many workers show as `THREADPOOL` waits.

## Remember for the exam
- Blocking finder: `sys.dm_exec_requests` (`blocking_session_id`) + `sys.dm_tran_locks`.
- Open transaction without COMMIT = classic cause of long blocking.
- Deadlock = SQL Server kills one **victim** (error 1205). Capture details with Extended Events.
- **RCSI / snapshot** use row versions: readers and writers do not block each other. RCSI is **on by default in Azure SQL Database**.
- Lock escalation at about **5,000 locks**; fix with smaller batches and indexes.
- `PAGEIOLATCH` = disk. `PAGELATCH` = memory page contention (often tempdb). Do not confuse them.
- `RESOURCE_SEMAPHORE` = memory grant. `SOS_SCHEDULER_YIELD` = CPU. `CXPACKET` = parallelism.
- `LOG_RATE_GOVERNOR` = Azure log write limit → scale up or reduce writes. Error **10928** = worker limit.

## Quick check
1. Readers in your SQL Server database are blocked by writers all day. You want readers to stop waiting without code changes. What do you enable?
   <details><summary>Answer</summary>READ_COMMITTED_SNAPSHOT (RCSI). Readers then use row versions instead of waiting for locks.</details>
2. Top wait is RESOURCE_SEMAPHORE. What does it mean?
   <details><summary>Answer</summary>Queries are waiting for memory grants. Tune large queries, fix bad estimates (statistics), or add memory.</details>
3. Which DMV column shows the session that is blocking a request?
   <details><summary>Answer</summary>blocking_session_id in sys.dm_exec_requests.</details>
