# Troubleshoot SQL Server Agent jobs

> **Exam skills:** Troubleshoot SQL Server Agent jobs, Configure job alerts and notifications
> **Reading time:** ~7 min

## The big idea
When a job fails, work through a short checklist. Is Agent running? What does the history say? Who does the job run as? Did the email go out? It is like a car that will not start: check fuel, then battery, then the key.

## 1. Is the Agent service running?
- **VM:** if the Agent service is stopped, jobs simply do not run. SSMS shows **"Agent XPs disabled"** next to SQL Server Agent.
- Set **Start mode = Automatic** in **SQL Server Configuration Manager**. Change the service account there too.
- Check SQL Server Agent **Error Logs** for start-up problems.

```sql
SELECT servicename, status_desc, startup_type_desc, service_account
FROM sys.dm_server_services;
```

- **Managed Instance:** Agent is **always running**. If a job does not run, look at the schedule, whether the job is enabled, and permissions.

## 2. Read the job history
| Object in `msdb` | What it holds |
|---|---|
| `dbo.sysjobs` | Job definitions (name, enabled, owner) |
| `dbo.sysjobsteps` | Step definitions (type, command, database, proxy) |
| `dbo.sysjobhistory` | Results. **`step_id = 0` is the overall job result.** A row is written **after** a step finishes. |
| `dbo.sysjobactivity` | Jobs **running right now** and the last run |

**`run_status`**: **0** Failed · **1** Succeeded · **2** Retry · **3** Canceled · **4** In progress.

```sql
SELECT j.name, h.step_id, h.step_name, h.run_date, h.run_time, h.message
FROM msdb.dbo.sysjobhistory AS h
JOIN msdb.dbo.sysjobs AS j ON j.job_id = h.job_id
WHERE h.run_status = 0                 -- failures only
ORDER BY h.run_date DESC, h.run_time DESC;

EXEC msdb.dbo.sp_help_jobhistory @job_name = N'NightlyBackup', @run_status = 0, @mode = N'FULL';
```

### "Old history is gone"
- Default limits: **1,000 rows in total** and **100 rows per job**. A busy job pushes old rows out quickly.
- **VM:** change this in **SQL Server Agent Properties → History**. Delete history by hand with `sp_purge_jobhistory`.
- **Managed Instance:** fixed at **1,000 / 100**. You **cannot change it**.
- The `message` column holds only **4,000 characters**. Long errors get cut off. Fix this with step output (next section).

## 3. Capture the full step output
On the job step's **Advanced** page (or `sp_update_jobstep`), send output to a **file** or a **table**:

| `@flags` | Effect |
|---|---|
| `0` / `2` | Overwrite / **append** to the output file |
| `8` / `16` | **Log to table** (`msdb.dbo.sysjobstepslogs`), overwrite / append |

```sql
EXEC msdb.dbo.sp_update_jobstep @job_name = N'NightlyIndexMaint', @step_id = 1,
     @output_file_name = N'D:\SQLLogs\IndexMaint.txt', @flags = 2;  -- append
```
On **Managed Instance** there is no file system access, so use **log to table**.

## 4. Who does the job run as?
| Problem | What you see | Fix |
|---|---|---|
| **Job owner left the company** (login dropped or disabled) | "Unable to determine if the owner ... has server access" | Make a **service login** or `sa` the owner: `sp_update_job @owner_login_name = N'sa'` |
| **Owner is not sysadmin** | T-SQL step runs as the owner and lacks rights | Grant the rights, or let a sysadmin own the job |
| **CmdExec / PowerShell / SSIS step denied** | Access denied to a file share or OS | Use a **proxy** with a **credential** that has the rights |
| **Proxy not usable** | Proxy not in the list, or step fails | Grant it: `sp_grant_proxy_to_subsystem` and `sp_grant_login_to_proxy` |

- **T-SQL steps never use proxies.** Only a **sysadmin** can create proxies.
- Non-sysadmin users need an Agent role in `msdb`: **SQLAgentUserRole**, **SQLAgentReaderRole** or **SQLAgentOperatorRole**. SQLAgentUserRole sees **only its own jobs**.

**Example:** A school's ETL job starts failing on Monday. Its owner was a DBA who left on Friday. His login was removed. Change the job owner to a service login and it works again.

## 5. Emails do not arrive
1. Check the Database Mail set-up from [lesson 3](<03 - SQL Server Agent jobs.md>): profile enabled in Agent and Agent restarted (VM), or profile named `AzureManagedInstance_dbmail_profile` plus outbound SMTP in the NSG (MI).
2. Check the **operator** is enabled and named on the job's **Notifications** page.
3. Look for send errors in `msdb.dbo.sysmail_event_log` and `sysmail_faileditems`.

## 6. Jobs and availability groups
- An **availability group (AG)** keeps copies of databases on several servers. But `msdb` is **not** in the AG, so **jobs do not fail over**.
- Create each job on **every replica**. Make the first step check whether this server is the primary:

```sql
IF ISNULL(sys.fn_hadr_is_primary_replica(N'SalesDb'), 0) <> 1
BEGIN
    PRINT 'Not primary - skipping.';
    RETURN;
END
```
- The function returns **1** on the primary, **0** on a secondary, **NULL** if the database is not in an AG.
- Exception: **SQL Server 2022+ contained AGs** have their own `msdb`, so their jobs follow the primary.
- **Managed Instance failover groups** do not copy `msdb` either. Re-create the jobs on the secondary.

## 7. Managed Instance Agent limits
| Supported on MI | Not supported on MI |
|---|---|
| T-SQL steps, SSIS steps, replication Snapshot / Log Reader / Distribution steps | **Proxies**, **Agent alerts**, Pager, NetSend, EventLog |
| Schedules, operators, **email** notifications | Turning Agent on/off; changing Agent properties (`sp_set_agent_properties`) |
| SQLAgent roles in `msdb` | CmdExec, **Idle-CPU** schedules, Merge / Queue Reader steps, SSAS, MSX/TSX, PowerShell Core |

- Microsoft Entra users must be **mapped directly to a login** (not only through a group) to manage jobs.

## Remember for the exam
- `run_status = 0` = failed. `step_id = 0` = the whole job's result.
- "Job running right now?" → `sysjobactivity`.
- "Older history missing" → **1,000 total / 100 per job** limits. On MI you cannot change them.
- "Long error is cut off" → output **file** or **log to table**.
- "Job fails after an employee left" → change the **job owner**.
- "CmdExec step cannot reach a share" → **proxy + credential** (VM only).
- "Job runs or fails on the AG secondary" → add **`sys.fn_hadr_is_primary_replica`** and create the job on every replica.
- MI: **no proxies, no alerts**, mail profile **`AzureManagedInstance_dbmail_profile`**.

## Quick check
1. A job's history shows only the last few days. Where is the rest?
   <details><summary>Answer</summary>It was removed by the history limits (1,000 rows total, 100 per job). On a VM you can raise them; on MI you cannot.</details>
2. A backup job runs on both AG replicas and fails on the secondary. What do you add?
   <details><summary>Answer</summary>A first step that checks sys.fn_hadr_is_primary_replica and exits if the server is not the primary.</details>
3. On Managed Instance you try to create a proxy for a job step. What happens?
   <details><summary>Answer</summary>It is not supported. MI does not support SQL Agent proxies.</details>
