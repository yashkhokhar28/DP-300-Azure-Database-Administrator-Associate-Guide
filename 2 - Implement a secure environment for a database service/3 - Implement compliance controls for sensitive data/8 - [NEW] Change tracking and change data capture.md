## 🔁 **Implement Change Data Tracking: Two Built-in Features**

- DP-300 skill: **"Implement change data tracking"**. SQL Server / Azure SQL ship **two** built-in features that tell an application **which DML changes** (insert, update, delete) happened to user tables:
  - **Change Tracking (CT)** – lightweight: *"which rows changed?"*
  - **Change Data Capture (CDC)** – detailed: *"what exactly changed, and what were the old and new values?"*
- Both replace custom triggers + timestamp columns + side tables. Both can be enabled on the **same database** at the same time.

---

## ⚖️ **Change Tracking vs Change Data Capture**

| Feature                      | **Change Tracking (CT)**                                          | **Change Data Capture (CDC)**                                            |
| ---------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------ |
| Mechanism                    | **Synchronous** – recorded in the same transaction as the DML      | **Asynchronous** – capture process reads the **transaction log**          |
| What's recorded              | **Primary key** of changed rows + operation (I/U/D) + optionally which columns changed | **Full column data** of changed rows (insert/delete images, update **before and after** images) |
| History / intermediate values | ❌ **Net change only** (one row per changed row since a version)  | ✅ **Every change** (historical)                                          |
| Latest data                  | Join back to the **source table** on the PK                       | Stored in the change table itself                                         |
| Storage                      | Small internal side tables                                        | Change tables `cdc.<capture_instance>_CT` in the **same database**        |
| Default retention            | **2 days** (`CHANGE_RETENTION`, `AUTO_CLEANUP`)                   | **3 days** (4320 minutes) via cleanup job / scheduler                     |
| Primary key required?        | ✅ **Yes** – table must have a PK                                  | ❌ No (only needed for **net changes** function: PK or unique index)       |
| Availability of changes      | **Immediately** after commit                                      | After small **latency** (capture must scan the log)                       |
| Overhead                     | Low DML overhead, minimal storage                                 | Higher: log scan, more storage, can **hold log truncation**               |
| Typical use                  | **Sync** apps (one-way cache refresh, **two-way/offline sync**, conflict detection) | **ETL / incremental loads** into a data warehouse, event streaming |

---

## 🌐 **Platform Support**

| Platform                     | Change Tracking | CDC                                                                                   |
| ---------------------------- | --------------- | ------------------------------------------------------------------------------------- |
| **SQL Server** (incl. Azure VM) | ✅            | ✅ **Standard/Enterprise** editions; capture & cleanup run as **SQL Server Agent jobs**  |
| **Azure SQL Managed Instance**  | ✅            | ✅ Same as SQL Server – uses **SQL Agent** jobs                                         |
| **Azure SQL Database**          | ✅             | ✅ **vCore: any service tier** (single DBs & elastic pools). **DTU: S3 and above only** – Basic, S0, S1, S2 ❌. Uses the **CDC scheduler** (no SQL Agent) |

**Azure SQL Database CDC specifics:**

- **CDC scheduler** runs **capture every 20 seconds** and **cleanup every hour** – frequency **can't be customized** (`pollinginterval`/`continuous` don't apply).
- Can still tune `maxtrans` / `maxscans` with `sys.sp_cdc_change_job`; `cdc.cdc_jobs` lives in the **`cdc` schema** (not `msdb`). Manual runs: `sys.sp_cdc_scan`, `sys.sp_cdc_cleanup_change_table`.
- Enabling requires **`db_owner`** (on SQL Server/MI, `sp_cdc_enable_db` requires **sysadmin**).
- **Serverless** DB **paused** → CDC doesn't run (and CDC doesn't block auto-pause).
- Enabling CDC **disables ADR's aggressive log truncation** → watch log usage; consider more vCores or **Hyperscale**.
- After **failover** (local or geo), capture/cleanup resume automatically on the new primary.
- Monitor with **`sys.dm_cdc_log_scan_sessions`** and **`sys.dm_cdc_errors`**. **No SLA** on change latency.

---

## 🛠️ **Enable Change Tracking (T-SQL)**

```sql
-- 1. Database level (recommended: snapshot isolation for consistent reads)
ALTER DATABASE SalesDB SET ALLOW_SNAPSHOT_ISOLATION ON;
ALTER DATABASE SalesDB
SET CHANGE_TRACKING = ON (CHANGE_RETENTION = 2 DAYS, AUTO_CLEANUP = ON);

-- 2. Table level (table must have a PRIMARY KEY)
ALTER TABLE dbo.Customers
ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON);   -- default OFF (extra storage)

-- Check configuration
SELECT * FROM sys.change_tracking_databases;
SELECT * FROM sys.change_tracking_tables;

-- Disable: tables first, then the database
ALTER TABLE dbo.Customers DISABLE CHANGE_TRACKING;
ALTER DATABASE SalesDB SET CHANGE_TRACKING = OFF;
```

---

## 🔍 **Query Change Tracking**

```sql
DECLARE @last_sync_version BIGINT = 1050;            -- persisted by the app after the last sync

-- Validate the version isn't older than retention (else: full re-initialization)
IF @last_sync_version < CHANGE_TRACKING_MIN_VALID_VERSION(OBJECT_ID('dbo.Customers'))
    THROW 50001, 'Sync version too old - reinitialize', 1;

SELECT c.CustomerID, c.SYS_CHANGE_OPERATION,           -- I / U / D
       c.SYS_CHANGE_VERSION, cu.Name, cu.Email          -- latest data comes from the base table
FROM CHANGETABLE(CHANGES dbo.Customers, @last_sync_version) AS c
LEFT JOIN dbo.Customers AS cu ON cu.CustomerID = c.CustomerID;   -- LEFT JOIN keeps deleted rows

SELECT CHANGE_TRACKING_CURRENT_VERSION();              -- store for the next sync
```

- `CHANGETABLE(VERSION ...)` returns the latest version of a **single row** – used for **conflict detection** in two-way sync.
- Needs **`VIEW CHANGE TRACKING`** on the table + `SELECT` on PK columns.
- ⚠️ `TRUNCATE TABLE` isn't tracked row-by-row – it **resets** the table's CT version info (clients must re-sync).

---

## 🛠️ **Enable Change Data Capture (T-SQL)**

```sql
-- 1. Database level (creates cdc schema + cdc user; a schema/user named "cdc" must not already exist)
EXEC sys.sp_cdc_enable_db;
SELECT name, is_cdc_enabled FROM sys.databases;

-- 2. Table level -> creates capture instance dbo_Orders, change table cdc.dbo_Orders_CT
EXEC sys.sp_cdc_enable_table
    @source_schema        = N'dbo',
    @source_name          = N'Orders',
    @role_name            = N'cdc_reader',            -- gating role (NULL = no gating role)
    @captured_column_list = N'OrderID, Status, Amount',
    @supports_net_changes = 1;                         -- needs PK or @index_name

SELECT name, is_tracked_by_cdc FROM sys.tables;

-- Disable
EXEC sys.sp_cdc_disable_table @source_schema = N'dbo', @source_name = N'Orders',
                              @capture_instance = N'dbo_Orders';
EXEC sys.sp_cdc_disable_db;
```

---

## 🔍 **Query CDC Changes**

```sql
DECLARE @from_lsn BINARY(10) = sys.fn_cdc_get_min_lsn('dbo_Orders'),
        @to_lsn   BINARY(10) = sys.fn_cdc_get_max_lsn();

-- Every change; 'all update old' returns both before (3) and after (4) images for updates
SELECT __$start_lsn, __$operation, OrderID, Status, Amount
FROM cdc.fn_cdc_get_all_changes_dbo_Orders(@from_lsn, @to_lsn, N'all update old');

-- Only the final state per row in the range (requires @supports_net_changes = 1)
SELECT * FROM cdc.fn_cdc_get_net_changes_dbo_Orders(@from_lsn, @to_lsn, N'all');
```

- `__$operation`: **1** = delete, **2** = insert, **3** = update (before), **4** = update (after).
- Map times to LSNs with `sys.fn_cdc_map_time_to_lsn`. LSN range outside the validity interval → error.
- Max **2 capture instances per table** (used to handle schema changes without losing data).

---

## 🧹 **Cleanup & Retention**

| Feature | How cleanup works                                                                                                     |
| ------- | --------------------------------------------------------------------------------------------------------------------- |
| **CT**  | Background **auto cleanup** removes info older than `CHANGE_RETENTION`; set `AUTO_CLEANUP = OFF` to pause during sync problems. Retention must be ≥ longest gap between syncs |
| **CDC** | **Cleanup job** (SQL Server/MI: Agent job daily at 2 AM, 4320 min) or **scheduler** (SQL DB: hourly) deletes expired rows |

```sql
-- SQL Server / MI: reduce CDC retention to 2 days (minutes), then restart the job
EXEC sys.sp_cdc_change_job @job_type = N'cleanup', @retention = 2880;
EXEC sys.sp_cdc_stop_job  @job_type = N'cleanup';
EXEC sys.sp_cdc_start_job @job_type = N'cleanup';
```

- ⚠️ With CDC, the **log can't truncate** (even in SIMPLE recovery) until the capture process has harvested changes → stopped capture job = **growing log**.
- Restoring a CDC database to **another server** disables CDC unless you use **`RESTORE ... WITH KEEP_CDC`**.

---

## 🧭 **CT vs CDC vs Temporal Tables vs Auditing vs Ledger**

| Feature                 | Answers the question                         | Stores old values?          | Who/when (user identity)? | Tamper-evident? | Typical exam scenario                                  |
| ----------------------- | -------------------------------------------- | --------------------------- | ------------------------- | --------------- | ------------------------------------------------------ |
| **Change Tracking**     | *Which rows changed since my last sync?*     | ❌ (PK + operation only)     | ❌                         | ❌               | Mobile/offline **sync**, cache refresh, lowest overhead |
| **Change Data Capture** | *What were all inserts/updates/deletes, with values?* | ✅ before & after (log-based, retention-limited) | ❌ | ❌         | **Incremental ETL** to a data warehouse                 |
| **Temporal tables**     | *What did this row look like at time X?*     | ✅ full row versions in a **history table** (`FOR SYSTEM_TIME AS OF`) | ❌ (period times only) | ❌ | Point-in-time analysis, **recover from accidental changes**, SCD |
| **Auditing**            | *Who did what (incl. SELECT/logins) and when?* | ❌ (statement text, not row versions) | ✅                  | ❌               | **Compliance**: track access/activity to storage / Log Analytics / Event Hubs |
| **Ledger**              | *Can I prove data wasn't tampered with?*     | ✅ (updatable ledger history) | ✅ (transaction principal) | ✅ **SHA-256 hash chain + digests** | Prove **integrity** to auditors/partners       |

---

## ✅ **Summary**

- **CT** = synchronous, PK + operation only, net changes, **PK required**, 2-day default retention, `CHANGETABLE` + `CHANGE_TRACKING_CURRENT_VERSION()`.
- **CDC** = asynchronous log reader, full before/after data in `cdc.*_CT` tables, 3-day default retention, `cdc.fn_cdc_get_all_changes_<instance>` / `..._net_changes_...`.
- **CDC engine:** SQL Server & MI → **SQL Server Agent** jobs; Azure SQL DB → **CDC scheduler** (20 s capture / hourly cleanup, not configurable).
- **Azure SQL DB CDC tiers:** any **vCore** tier; **DTU S3+** only.

---

## 🎯 **Exam Tips / Traps**

- 🎯 "App needs to know **which rows changed** to sync a mobile/offline client with **minimal overhead**" → **Change Tracking**.
- 🎯 "ETL must load **every insert/update/delete with column values** incrementally" → **CDC**.
- 🎯 "Enable CDC on an Azure SQL Database in **S0/S1/S2/Basic**" → ❌ not supported → scale to **S3+** or **vCore**.
- 🎯 "CDC change tables stop filling on SQL Server / MI" → check **SQL Server Agent** is running (Azure SQL DB has no Agent – scheduler is built in).
- 🎯 "Transaction log keeps growing on a CDC-enabled DB" → capture not keeping up / stopped; log truncation is held.
- ⚠️ Trap: Enabling CT on a table **without a primary key** fails. CDC doesn't need a PK unless you want **net changes**.
- ⚠️ Trap: CT does **not** give you old values or *who* made the change – for history use **temporal tables**, for *who* use **auditing**, for *proof of integrity* use **ledger**.
- ⚠️ Trap: Enable at the **database** level first, then per **table** – for both features. For CT, disable tables **before** the database.
- ⚠️ Trap: CDC on SQL Server requires **Standard or Enterprise** edition (not Express/Web).

> Source: https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/track-data-changes-sql-server · https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-tracking-sql-server · https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/enable-and-disable-change-tracking-sql-server · https://learn.microsoft.com/en-us/sql/relational-databases/system-functions/changetable-transact-sql · https://learn.microsoft.com/en-us/sql/t-sql/statements/alter-table-transact-sql · https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-data-capture-sql-server · https://learn.microsoft.com/en-us/azure/azure-sql/database/change-data-capture-overview · https://learn.microsoft.com/en-us/sql/relational-databases/system-functions/cdc-fn-cdc-get-all-changes-capture-instance-transact-sql · https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sys-sp-cdc-change-job-transact-sql · https://learn.microsoft.com/en-us/azure/azure-sql/database/features-comparison · https://learn.microsoft.com/en-us/sql/relational-databases/tables/temporal-tables
