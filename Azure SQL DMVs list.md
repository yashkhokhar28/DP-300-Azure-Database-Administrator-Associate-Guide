# DP-300 DMV Playbook (Azure SQL + Managed Instance)

**Permissions you’ll need**
- Azure SQL Database: `VIEW DATABASE STATE`
- Managed Instance / SQL Server: `VIEW SERVER STATE` (for instance-wide DMVs)

---

## 1) Plan & Implement Data Platform Resources

| DMV | Scope | What it tells you | Typical exam scenario |
|---|---|---|---|
| `sys.dm_db_resource_stats` | Azure SQL DB | 15-sec samples of CPU, data I/O, log I/O, storage | Decide if spikes justify scaling a DB or moving it to a pool |
| `sys.resource_stats` | Azure master (PaaS) | 5-min aggregates for all DBs in a server | Capacity planning across multiple DBs over days |
| `sys.elastic_pool_resource_stats` | Elastic Pool | CPU/IO/memory use per pool | Right-size the pool vs. single DB compute |
| `sys.dm_io_virtual_file_stats(NULL,NULL)` | DB/Instance | File-level read/write latency (stall_ms) | Pick premium SSD tier or add files/FGs to reduce I/O stalls |
| `sys.dm_db_file_space_usage` | DB | Data/log/`version_store` usage (esp. tempdb in MI) | Detect row-version growth with long transactions |
| `sys.dm_db_partition_stats` | DB | Row and page counts by object/partition | Estimate storage & rebuild cost before index work |

**Quick sizing query**
```sql
SELECT end_time, avg_cpu_percent, avg_log_write_percent, avg_data_io_percent
FROM sys.dm_db_resource_stats
ORDER BY end_time DESC;
````

---

## 2) Implement a Secure Environment

| DMV                                                | Scope       | What it tells you                           | Typical exam scenario                                   |
| -------------------------------------------------- | ----------- | ------------------------------------------- | ------------------------------------------------------- |
| `sys.dm_database_encryption_keys`                  | DB          | TDE state, encryptor type, thumbprint       | Verify TDE enabled after migration/restore              |
| `sys.dm_exec_connections`                          | DB/Instance | Client IP, protocol, encryption             | Confirm connections are encrypted; trace suspicious IPs |
| `sys.dm_exec_sessions`                             | DB/Instance | Who’s connected, program\_name, login\_time | Hunt risky logins; idle sessions policy enforcement     |
| `sys.dm_exec_input_buffer(session_id, request_id)` | DB/Instance | Last statement executed by a session        | Forensics on what a compromised login ran               |
| `sys.dm_cryptographic_provider_properties`\*       | MI/SQL      | Extensible key mgmt provider state          | Validate EKM provider (HSM) health *(MI/SQL only)*      |

---

## 3) Monitor, Configure, and **Optimize** Database Resources

### 3a) Sessions, Requests, Waits

| DMV                            | Scope        | What it tells you               | Exam scenario                              |
| ------------------------------ | ------------ | ------------------------------- | ------------------------------------------ |
| `sys.dm_exec_sessions`         | DB/Instance  | All sessions (user/system)      | Baseline activity; spot noisy apps         |
| `sys.dm_exec_connections`      | DB/Instance  | Network details & encrypt state | Encrypted/unencrypted, IP source           |
| `sys.dm_exec_requests`         | DB/Instance  | Currently running requests      | Find blockers, long runners, waits         |
| `sys.dm_exec_requests_history` | Azure SQL DB | **Historical** requests (PaaS)  | Investigate a spike after it’s gone        |
| `sys.dm_db_wait_stats`         | Azure SQL DB | DB-scoped waits (PaaS)          | Identify top waits without instance access |
| `sys.dm_os_wait_stats`         | MI/SQL       | Instance waits                  | Root-cause on Managed Instance/VM          |

**Top wait types now**

```sql
SELECT TOP 10 wait_type, wait_time_ms, signal_wait_time_ms
FROM sys.dm_db_wait_stats
ORDER BY wait_time_ms DESC;
```

### 3b) Query Text, Plans & Caching

| DMV                                   | Scope       | What it tells you                | Exam scenario                       |
| ------------------------------------- | ----------- | -------------------------------- | ----------------------------------- |
| `sys.dm_exec_sql_text(sql_handle)`    | DB/Instance | Full SQL text                    | Tie a plan/req to exact text        |
| `sys.dm_exec_query_plan(plan_handle)` | DB/Instance | Actual cached plan XML           | Diagnose scans, key lookups, spills |
| `sys.dm_exec_query_stats`             | DB/Instance | CPU/reads/writes per cached plan | Find top CPU or I/O consumers       |
| `sys.dm_exec_cached_plans`            | DB/Instance | Cache use & reuse counts         | Detect ad-hoc plan cache bloat      |

**Top CPU queries**

```sql
SELECT TOP 10 total_worker_time/1000 AS total_cpu_ms,
       execution_count, DB_NAME(t.dbid) AS dbname,
       LEFT(st.text,4000) AS sample_sql
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
CROSS APPLY sys.dm_exec_text_query_plan(qs.plan_handle, qs.statement_start_offset, qs.statement_end_offset) t
ORDER BY total_worker_time DESC;
```

### 3c) Indexes & Missing Index Hints

| DMV / DMF                                 | Scope | What it tells you           | Exam scenario                             |
| ----------------------------------------- | ----- | --------------------------- | ----------------------------------------- |
| `sys.dm_db_index_usage_stats`             | DB    | Seeks/scans/lookups/updates | Drop unused; validate benefit of an index |
| `sys.dm_db_index_operational_stats` (DMF) | DB    | Latch/lock/io per index     | Prove an index is hot or contended        |
| `sys.dm_db_missing_index_details`         | DB    | Columns for potential index | Identify helpful indexes after workload   |
| `sys.dm_db_missing_index_group_stats`     | DB    | Impact estimate per group   | Prioritize which index to create first    |
| `sys.dm_db_partition_stats`               | DB    | Row counts/size per index   | Estimate rebuild cost/fillfactor          |

### 3d) TempDB & Memory Pressure

| DMV                             | Scope | What it tells you             | Exam scenario                            |
| ------------------------------- | ----- | ----------------------------- | ---------------------------------------- |
| `sys.dm_db_session_space_usage` | DB    | Tempdb pages per session      | Identify sessions causing spills/sorts   |
| `sys.dm_db_task_space_usage`    | DB    | Tempdb by task within session | Pinpoint the exact request causing bloat |
| `sys.dm_db_log_space_usage`     | DB    | Log used vs. total            | Large transactions throttling log        |

### 3e) Resource Governance & Auto Tuning (Azure)

| DMV                                  | Scope              | What it tells you                | Exam scenario                                         |
| ------------------------------------ | ------------------ | -------------------------------- | ----------------------------------------------------- |
| `sys.dm_user_db_resource_governance` | Azure SQL DB       | Worker/I/O caps per service tier | Confirm workload is capped per sku                    |
| `sys.dm_db_tuning_recommendations`   | Azure SQL DB       | Auto-tuning suggestions & state  | Approve/verify CREATE/DROP index and plan corrections |
| `sys.dm_exec_distributed_tasks`      | Hyperscale/Elastic | Fan-out task state               | Track long distributed operations                     |

---

## 4) Configure and Manage **Automation** of Tasks

> Azure SQL DB uses **Automatic Tuning** and platform jobs; MI can use SQL Agent.

| DMV                                  | Scope              | What it tells you                       | Exam scenario                              |
| ------------------------------------ | ------------------ | --------------------------------------- | ------------------------------------------ |
| `sys.dm_db_tuning_recommendations`   | Azure SQL DB       | Pending/applied tuning actions          | Verify auto-tuning fixed a regression      |
| `sys.dm_operation_status`            | Azure SQL (master) | Long-running ops: restore/copy/reconfig | Track restore progress; tenant copy status |
| `sys.dm_exec_background_job_queue`\* | MI/SQL (rare)      | Internal tasks                          | Deep dive (not common on exam)             |

---

## 5) Plan & Configure **HA/DR**

### Azure SQL Database (PaaS)

| DMV                                  | Scope        | What it tells you                      | Exam scenario                                          |
| ------------------------------------ | ------------ | -------------------------------------- | ------------------------------------------------------ |
| `sys.dm_geo_replication_link_status` | Azure SQL DB | AG-like geo-rep link health/lag        | Validate active geo-replication health before failover |
| `sys.dm_continuous_copy_status`      | Azure SQL DB | Copy/replication state                 | Monitor seeding/catch-up after creating sec.           |
| `sys.dm_database_backups`            | Azure SQL DB | Point-in-time backups metadata         | Confirm PITR window for compliance                     |
| `sys.dm_database_backup_lineage`     | Azure SQL DB | Backup chain & lineage across restores | Ensure restore chain intact across regions             |

### Managed Instance / SQL Server (AG, FCI)

| DMV                                       | Scope  | What it tells you                | Exam scenario                                      |
| ----------------------------------------- | ------ | -------------------------------- | -------------------------------------------------- |
| `sys.dm_hadr_availability_group_states`   | MI/SQL | AG role & health                 | Is this node primary/secondary?                    |
| `sys.dm_hadr_availability_replica_states` | MI/SQL | Replica connection/role          | Validate synchronous commit and failover readiness |
| `sys.dm_hadr_database_replica_states`     | MI/SQL | DB sync state, redo, LSNs        | Measure redo lag before a planned failover         |
| `sys.dm_io_virtual_file_stats`            | MI/SQL | I/O latency (redo queue insight) | Diagnose slow redo due to storage issues           |

**Geo-rep status (PaaS)**

```sql
SELECT link_guid, partner_server, replication_state_desc, last_replication, secondary_commit_latency
FROM sys.dm_geo_replication_link_status;
```

---

## “Find X Fast” — Mini Playbook

* **Top CPU query right now** → `dm_exec_requests` + `dm_exec_sql_text`
* **Top CPU historically (cached)** → `dm_exec_query_stats`
* **Blocking chain** → `dm_exec_requests` (waits) + `dm_tran_locks`
* **Tempdb hog** → `dm_db_session_space_usage`
* **Missing indexes** → `dm_db_missing_index_*`
* **DTU/vCore pressure** → `dm_db_resource_stats` / `elastic_pool_resource_stats`
* **Geo-rep health** → `dm_geo_replication_link_status`
* **TDE enabled?** → `dm_database_encryption_keys`

---

## Rapid Labs (15–20 min each)

1. **CPU spike RCA**

   * Run: `dm_db_resource_stats` → time window
   * Correlate: `dm_exec_query_stats` (TOP 5 total\_worker\_time)
   * Action: check plan via `dm_exec_query_plan`, propose index

2. **Tempdb meltdown**

   * Run: `dm_db_session_space_usage` ordered by `internal_objects_alloc_page_count`
   * Grab query with `dm_exec_input_buffer`

3. **HA drill**

   * PaaS: `dm_geo_replication_link_status` (latency, state)
   * MI: `dm_hadr_*` (sync state) + `dm_io_virtual_file_stats` (redo stall)

---

## Edge Notes (Exam-bait)

* **Azure SQL DB** does **not** expose many `sys.dm_os_*` instance DMVs — use `sys.dm_db_wait_stats` instead of `sys.dm_os_wait_stats`.
* **Auto-tuning** evidence lives in `sys.dm_db_tuning_recommendations` and the `Automatic Tuning` catalog views.
* **History** on PaaS: `sys.resource_stats` (5-min) vs `dm_db_resource_stats` (15-sec).
* **Backups on PaaS** are platform-managed—know `dm_database_backups` & `dm_database_backup_lineage` for restore questions.
