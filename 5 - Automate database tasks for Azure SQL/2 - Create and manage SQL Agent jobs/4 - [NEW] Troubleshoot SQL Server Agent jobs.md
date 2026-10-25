### ✅ **Troubleshoot SQL Server Agent Jobs**

---

## 🗺️ **Where Does SQL Agent Exist?**

| Platform | SQL Server Agent? | Notes |
| --- | --- | --- |
| **SQL Server on Azure VM** | ✅ Full Agent | You manage the service, start mode, service account, proxies, alerts |
| **Azure SQL Managed Instance** | ✅ Built-in, **always running** | Can't stop/start it; settings are **read-only**; several features missing (see below) |
| **Azure SQL Database** | ❌ **No Agent** | Use **Elastic Jobs** (or Azure Automation / Logic Apps / Functions) |

---

## 🔌 **1. Is the Agent Service Running?**

On SQL Server (VM) — jobs simply don't run if the Agent service is stopped. SSMS shows **"Agent XPs disabled"** next to SQL Server Agent when it's stopped.

```sql
-- Service status, start mode and service account
SELECT servicename, status_desc, startup_type_desc, service_account, last_startup_time
FROM sys.dm_server_services;
```

- Set **Start mode = Automatic** (and change the service account) in **SQL Server Configuration Manager**; check SSMS → SQL Server Agent → **Error Logs** for startup failures.
- Agent service account must be **sysadmin** (job steps run as the Agent service account unless a proxy is used).
- **Managed Instance:** Agent **can't be enabled/disabled — always running**. If a job doesn't run, look elsewhere (schedule, job enabled, permissions).

---

## 📜 **2. Read the Job History**

| Object (in `msdb`) | Holds |
| --- | --- |
| `dbo.sysjobs` | Job definitions (name, enabled, owner_sid) |
| `dbo.sysjobsteps` | Step definitions (subsystem, command, database, output file, proxy) |
| `dbo.sysjobhistory` | Outcome rows — **`step_id = 0` is the overall job outcome**; written **after a step completes** |
| `dbo.sysjobactivity` | Current/last session activity — use it for **jobs running right now** |
| `dbo.sp_help_jobhistory` | Proc wrapper (`@run_status = 0` for failures, `@mode = 'FULL'`) |

**`run_status`**: `0` Failed · `1` Succeeded · `2` Retry · `3` Canceled · `4` In progress. `run_date` = `yyyyMMdd` int, `run_time`/`run_duration` = `HHmmss` int.

```sql
-- Failed jobs/steps in the last 24 hours
SELECT  j.name                     AS job_name,
        h.step_id, h.step_name,
        msdb.dbo.agent_datetime(h.run_date, h.run_time) AS run_start,   -- undocumented helper
        h.sql_message_id, h.sql_severity,
        h.message
FROM msdb.dbo.sysjobhistory AS h
JOIN msdb.dbo.sysjobs       AS j ON j.job_id = h.job_id
WHERE h.run_status = 0
  AND msdb.dbo.agent_datetime(h.run_date, h.run_time) >= DATEADD(HOUR, -24, GETDATE())
ORDER BY run_start DESC;

-- Documented alternative
EXEC msdb.dbo.sp_help_jobhistory @job_name = N'NightlyBackups', @run_status = 0, @mode = N'FULL';
```

### 🗑️ **History retention (why "the history is gone")**

- Defaults: **1,000 rows total** and **100 rows per job** → busy jobs quickly push old rows out.
- SQL Server: change in **SQL Agent Properties → History** (max rows / per-job rows, or *Remove agent history older than…*). Purge manually with `msdb.dbo.sp_purge_jobhistory @job_name = …, @oldest_date = …`.
- **Managed Instance: fixed at 1,000 total / 100 per job — can't change** (`sp_set_agent_properties` not supported).
- `message` column is only **nvarchar(4000)** — long errors get truncated → use step logging ⬇️.

---

## 📝 **3. Capture Full Step Output**

`sp_add_jobstep` / `sp_update_jobstep` **`@flags`** and **`@output_file_name`** (SSMS: Job Step → **Advanced**):

| `@flags` | Effect |
| --- | --- |
| `0` / `2` | Overwrite / **append** output file |
| `4` | Write T-SQL step output to **step history** |
| `8` / `16` | **Log to table** (`msdb.dbo.sysjobstepslogs`) — overwrite / append |
| `32` | Write **all output** to job history |

```sql
EXEC msdb.dbo.sp_update_jobstep
     @job_name = N'NightlyIndexMaint', @step_id = 1,
     @output_file_name = N'D:\SQLLogs\IndexMaint_$(ESCAPE_SQUOTE(STRTDT)).txt',
     @flags = 2;   -- append
```

> MI has no access to the file system/shares → prefer **log to table**.

---

## 🔐 **4. Security Context: Owner, Proxies, Credentials**

| Problem | Symptom | Fix |
| --- | --- | --- |
| **Orphaned/disabled job owner** (e.g., leaver's Windows login dropped) | Job fails: *unable to determine if the owner … has server access* | Change owner to a **dedicated service login** or `sa`: `EXEC msdb.dbo.sp_update_job @job_name=N'…', @owner_login_name=N'sa';` |
| **Non-sysadmin owner** | T-SQL step runs **as the owner login**; `@database_user_name` ignored | Grant needed perms, or have a sysadmin own the job |
| **CmdExec/PowerShell/SSIS step lacks rights** | Access denied to file share / OS | Use a **proxy** mapped to a **credential** with rights |
| **Proxy not granted** | Step fails / proxy not selectable | `sp_grant_proxy_to_subsystem` + `sp_grant_login_to_proxy` |

```sql
USE msdb;
CREATE CREDENTIAL FileShareCred WITH IDENTITY = N'CONTOSO\svc-sqlfiles', SECRET = N'<strong password>';
EXEC dbo.sp_add_proxy @proxy_name = N'FileShareProxy', @credential_name = N'FileShareCred', @enabled = 1;
EXEC dbo.sp_grant_proxy_to_subsystem @proxy_name = N'FileShareProxy', @subsystem_name = N'CmdExec';
EXEC dbo.sp_grant_login_to_proxy    @proxy_name = N'FileShareProxy', @login_name = N'CONTOSO\etl-team';
```

- **T-SQL steps never use proxies** (they run as owner / `@database_user_name`). Only **sysadmin** creates proxies; the credential's Windows account needs **"Access this computer from the network"**.
- Non-sysadmins need **SQLAgentUserRole / ReaderRole / OperatorRole** in `msdb` (SQLAgentUserRole sees **only its own jobs**' history).

---

## ✉️ **5. Notifications Not Arriving**

- **Database Mail** enabled (`Database Mail XPs`) and a **profile** configured.
- **SQL Server:** Agent Properties → **Alert System → Enable mail profile** + **restart Agent**.
- **Managed Instance:** profile **must be named `AzureManagedInstance_dbmail_profile`**; NSG must allow outbound SMTP port (25/587/465).
- Operator exists/enabled and job **Notifications** set (`@notify_level_email`: 1 success, 2 failure, 3 completion); check `msdb.dbo.sysmail_event_log` / `sysmail_faileditems` for send failures.

---

## 🔁 **6. Jobs and Availability Groups**

- `msdb` is **not** in the AG → jobs **don't fail over** → create jobs on **every replica**, and make them **primary-aware**:

```sql
-- Step 1 of every AG-aware job
IF ISNULL(sys.fn_hadr_is_primary_replica(N'SalesDb'), 0) <> 1
BEGIN
    PRINT 'Not primary replica - skipping.';
    RETURN;
END
-- ...real work here...
```

- `sys.fn_hadr_is_primary_replica` returns **1** primary, **0** secondary, **NULL** if DB not in an AG (hence `ISNULL`). Needs **VIEW SERVER STATE**.
- **SQL Server 2022+ contained AGs** have their own `msdb` → jobs created via the **contained AG listener** replicate and **run only on the primary**.

---

## ☁️ **7. Managed Instance Agent Differences**

| Supported | Not supported |
| --- | --- |
| T-SQL steps, **SSIS** steps, replication **Snapshot / Log Reader / Distribution** | **Proxies**, **Alerts**, Pager, NetSend, EventLog |
| Schedules, operators, **email** notifications | Enable/disable Agent; changing Agent properties (`sp_set_agent_properties`) |
| SQLAgent fixed DB roles (may need explicit `GRANT EXECUTE` on `xp_sqlagent_*` in `master`, error 229) | **Idle-CPU** schedules, Merge / Queue Reader steps, SSAS, MSX/TSX multi-server jobs, scripts from disk, PowerShell Core |

- Docs conflict on **CmdExec**: T-SQL-differences page says **"Command shell isn't yet supported"** → treat OS-level steps as not available on MI.
- Entra users must be **directly mapped to a login** (not via group) to manage jobs. **Failover groups** don't replicate `msdb` → recreate jobs on the secondary.

---

## ✅ **Summary**

| Check | Where |
| --- | --- |
| Agent running? | `sys.dm_server_services`, Config Manager (VM only) |
| What failed? | `sysjobhistory` (`run_status = 0`), `sp_help_jobhistory` |
| Running now? | `sysjobactivity` |
| Full error text | Step output file / log to table (`@flags`) |
| Permissions | Job owner, proxy + credential, msdb Agent roles |
| No email | Database Mail profile (MI: `AzureManagedInstance_dbmail_profile`) |
| AG | Job on every replica + `fn_hadr_is_primary_replica` |
| Azure SQL DB | No Agent → **Elastic Jobs** (history in `jobs.job_executions`, auto-purged after **45 days**) |

---

## 🎯 **Exam Tips / Traps**

- **"Schedule a T-SQL job in Azure SQL Database"** → **Elastic job agent**, not SQL Agent.
- **MI: Agent always on**; **no proxies, no alerts**; mail profile name **`AzureManagedInstance_dbmail_profile`**; history **fixed 1,000/100**.
- **Job history missing older runs** → history row limits (**1,000 total / 100 per job**).
- **Job fails after employee left** → **job owner** login removed → change owner.
- **CmdExec step can't reach a file share** → **proxy + credential** (VM only).
- **Job runs on both AG replicas / fails on secondary** → add **`sys.fn_hadr_is_primary_replica`** check; jobs aren't synced by AG (except contained AGs).
- **Long error message cut off** → configure **output file** or **log to table**.
- `step_id = 0` row = **job outcome**; filter `run_status = 0` for failures.

---

> Source: https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/job-automation-managed-instance, https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/transact-sql-tsql-differences-sql-server#sql-server-agent, https://learn.microsoft.com/en-us/sql/relational-databases/system-tables/dbo-sysjobhistory-transact-sql, https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-help-jobhistory-transact-sql, https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-purge-jobhistory-transact-sql, https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-add-jobstep-transact-sql, https://learn.microsoft.com/en-us/ssms/agent/resize-the-job-history-log, https://learn.microsoft.com/en-us/ssms/agent/create-a-sql-server-agent-proxy, https://learn.microsoft.com/en-us/sql/relational-databases/system-functions/sys-fn-hadr-is-primary-replica-transact-sql, https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-objects/sys-dm-server-services-transact-sql, https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/contained-availability-groups-overview, https://learn.microsoft.com/en-us/azure/azure-sql/database/elastic-jobs-overview
