# Change tracking and change data capture

> **Exam skills:** Implement change data tracking
> **Reading time:** ~10 min

## The big idea
Apps often need to know **what changed** in a table since last time. SQL Server and Azure SQL have two built-in features for this:
- **Change Tracking (CT)**: light. Answers "**which rows** changed?"
- **Change Data Capture (CDC)**: detailed. Answers "**what exactly** changed, with old and new values?"

Think of a school register. Change tracking is a sticky note saying "Rows 4 and 9 changed". CDC is a full diary: "Row 4: grade went from B to A at 10:02, then A to A+ at 11:15."

Both replace home-made triggers and timestamp columns. You can turn on both in the same database.

## Compare at a glance

| | **Change Tracking** | **Change Data Capture** |
|--|---------------------|-------------------------|
| How | **Synchronous**: recorded in the same transaction as the change | **Asynchronous**: a capture process reads the **transaction log** later |
| What is stored | **Primary key** of changed rows + operation (insert/update/delete) + optionally which columns changed | **Full column values**: inserted and deleted rows, and **before and after** values of updates |
| History | **Net change only** (latest state) | **Every** change |
| Latest data | Join back to the source table | In the change table |
| Primary key needed? | **Yes** | **No** (only for "net changes" queries) |
| Changes visible | Immediately after commit | After a short delay |
| Default retention | **2 days** | **3 days** (4,320 minutes) |
| Overhead | Low | Higher: log reading, more storage, can **hold the log** |
| Typical use | **Sync** apps: mobile/offline sync, cache refresh, two-way sync with conflict checks | **ETL**: load every change into a data warehouse |

**Example (CT):** A delivery driver app works offline. When it reconnects, it asks "which orders changed since version 1050?" and downloads just those rows.

**Example (CDC):** A nightly ETL job loads every insert, update and delete from the shop's `Orders` table into the data warehouse, including the old price before each change.

## Where they work

| Platform | Change Tracking | CDC |
|----------|-----------------|-----|
| SQL Server (incl. Azure VM) | Yes | Yes, **Standard or Enterprise** edition. Uses **SQL Server Agent** jobs |
| Azure SQL Managed Instance | Yes | Yes, uses **SQL Agent** jobs |
| Azure SQL Database | Yes | Yes: **any vCore tier**; **DTU S3 and above only** (not Basic, S0, S1, S2). Uses a built-in **CDC scheduler** (no SQL Agent) |

### CDC in Azure SQL Database
- The scheduler runs **capture every 20 seconds** and **cleanup every hour**. You **cannot change** these times.
- You can still tune `maxtrans` and `maxscans` with `sys.sp_cdc_change_job`. The job table `cdc.cdc_jobs` is in the **cdc schema** (not `msdb`).
- To enable it you need **db_owner**. (On SQL Server and MI, `sp_cdc_enable_db` needs **sysadmin**.)
- A **paused serverless** database does not run CDC.
- CDC turns off ADR's aggressive log truncation, so **watch log usage**. Consider more vCores or **Hyperscale**.
- After a failover, CDC carries on automatically on the new primary.
- Monitor with **`sys.dm_cdc_log_scan_sessions`** and **`sys.dm_cdc_errors`**.

## Turn on Change Tracking
Turn it on for the **database first**, then for each **table**.

```sql
ALTER DATABASE SalesDB SET ALLOW_SNAPSHOT_ISOLATION ON;   -- recommended
ALTER DATABASE SalesDB
SET CHANGE_TRACKING = ON (CHANGE_RETENTION = 2 DAYS, AUTO_CLEANUP = ON);

ALTER TABLE dbo.Customers
ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON);   -- table needs a PK
```

- `TRACK_COLUMNS_UPDATED` is **OFF** by default (it costs extra storage).
- Check with `sys.change_tracking_databases` and `sys.change_tracking_tables`.
- To turn off: disable **tables first**, then the database.

### Read changes

```sql
SELECT c.CustomerID, c.SYS_CHANGE_OPERATION, cu.Name
FROM CHANGETABLE(CHANGES dbo.Customers, @last_sync_version) AS c
LEFT JOIN dbo.Customers AS cu ON cu.CustomerID = c.CustomerID;
SELECT CHANGE_TRACKING_CURRENT_VERSION();   -- save for next time
```

- If `@last_sync_version` is older than `CHANGE_TRACKING_MIN_VALID_VERSION(...)`, the data has been cleaned up. The client must **reload everything**.
- `CHANGETABLE(VERSION ...)` gives the version of one row. It is used to **detect conflicts** in two-way sync.
- Readers need **VIEW CHANGE TRACKING** on the table and SELECT on the primary key columns.
- `TRUNCATE TABLE` is not tracked row by row. It **resets** tracking for the table, so clients must re-sync.
- Retention must be **longer than the longest gap between syncs**.

## Turn on Change Data Capture
Again: **database first**, then each **table**.

```sql
EXEC sys.sp_cdc_enable_db;   -- creates the cdc schema and cdc user

EXEC sys.sp_cdc_enable_table
    @source_schema = N'dbo', @source_name = N'Orders',
    @role_name = N'cdc_reader',            -- NULL = no gating role
    @supports_net_changes = 1;             -- needs a PK or unique index
```

- This makes a **capture instance** `dbo_Orders` and a change table **`cdc.dbo_Orders_CT`** in the same database.
- Check with `is_cdc_enabled` in `sys.databases` and `is_tracked_by_cdc` in `sys.tables`.
- A table can have at most **2 capture instances** (used to handle schema changes).
- Turn off with `sys.sp_cdc_disable_table` and `sys.sp_cdc_disable_db`.

### Read changes

```sql
DECLARE @from BINARY(10) = sys.fn_cdc_get_min_lsn('dbo_Orders'),
        @to   BINARY(10) = sys.fn_cdc_get_max_lsn();
SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_Orders(@from, @to, N'all update old');
SELECT * FROM cdc.fn_cdc_get_net_changes_dbo_Orders(@from, @to, N'all');
```

- **All changes** returns every change. **Net changes** returns only the final state of each row (needs `@supports_net_changes = 1`).
- `__$operation`: **1** = delete, **2** = insert, **3** = update (before), **4** = update (after).
- Convert times to LSNs (log sequence numbers: positions in the log) with `sys.fn_cdc_map_time_to_lsn`.

### Cleanup and the log
- SQL Server / MI: a cleanup **Agent job** runs daily at 2 AM and keeps 4,320 minutes by default. Change it with `sys.sp_cdc_change_job @job_type = N'cleanup', @retention = 2880;`, then stop and start the job.
- **The log cannot be truncated** until CDC has read the changes, even in SIMPLE recovery. A stopped capture job = a **growing log**.
- Restoring a CDC database to **another server** turns CDC off, unless you use **`RESTORE ... WITH KEEP_CDC`**.

## Which feature for which question?

| Feature | Answers | Old values? | Who did it? | Tamper-proof? |
|---------|---------|-------------|-------------|---------------|
| Change Tracking | Which rows changed since my last sync? | No | No | No |
| CDC | Every insert/update/delete, with values? | Yes, before and after | No | No |
| Temporal tables | What did this row look like at time X? | Yes, in a history table | No | No |
| Auditing | Who did what, and when (incl. SELECT and logins)? | No | **Yes** | No |
| Ledger | Can I prove nothing was tampered with? | Yes (updatable ledger) | Yes | **Yes** |

**Example:** A shop wants to see what a product's price was last March → **temporal table**. It wants to know which employee ran a query on salaries → **auditing**.

## Remember for the exam
- "Sync with **minimal overhead**, which rows changed" → **Change Tracking**.
- "Load **every change with column values** into a warehouse" → **CDC**.
- CT needs a **primary key**; CDC does not (except for net changes).
- Defaults: CT **2 days**; CDC **3 days**.
- Enable at the **database** level, then **table** level. For CT, disable **tables before** the database.
- CDC on Azure SQL Database: **vCore any tier** or **DTU S3+**. Not Basic/S0/S1/S2.
- CDC on SQL Server/MI uses **SQL Server Agent**. "CDC stopped filling" → check the Agent. Azure SQL Database uses a built-in scheduler (20 s capture, hourly cleanup, fixed).
- "Log keeps growing on a CDC database" → the capture is stopped or behind.
- CDC on SQL Server needs **Standard or Enterprise** edition.
- CT does not keep old values or the user's name. For history → **temporal tables**; for who → **auditing**; for proof → **ledger**.

## Quick check
1. You try to enable Change Tracking on a table and it fails. What is the likely cause?
   <details><summary>Answer</summary>The table has no primary key. Change Tracking requires one.</details>
2. Your Azure SQL Database is on the S1 tier and you need CDC. What must you do?
   <details><summary>Answer</summary>Scale to S3 or higher, or move to a vCore tier. CDC is not supported below S3 in the DTU model.</details>
3. On a Managed Instance, CDC change tables stopped getting new rows. What do you check first?
   <details><summary>Answer</summary>That SQL Server Agent is running, because CDC capture runs as an Agent job on MI and SQL Server.</details>
