# Monitor, test and troubleshoot HA/DR

> **Exam skills:** Plan a testing procedure for an HA/DR solution, Monitor an HA/DR solution, Troubleshoot an HA/DR solution
> **Reading time:** ~13 min

## The big idea
An HA/DR design is only a theory until you fail it over, measure it and fail it back. You **test** it with drills, **monitor** it so you see lag before it hurts, and **troubleshoot** it when replicas fall behind or failovers fail.

Think of a fire drill at a school. You ring the bell, time how long it takes to get everyone out, check nobody is missing, and fix the blocked door you found.

## Plan a testing procedure (DR drill)
| Step | What to do |
|---|---|
| 1. Scope | List every part: database, app tier, DNS, storage, logins, jobs, firewall/NSG rules |
| 2. Baseline | Record current lag and the target RPO/RTO |
| 3. Choose test type | **Planned** failover (safe in production) or **forced** (test environment only) |
| 4. Run | Run the failover. Note start and end times |
| 5. Validate | App connects through the **listener**, logins work, jobs run, reads and writes succeed |
| 6. Record | Actual RTO, actual RPO, problems, runbook fixes |
| 7. Fail back | Use a **planned** failover to return |

- **Planned failover** = sync first, **no data loss**. Use it for drills in production, moving regions and **failback**.
- **Forced failover** = switch now, **possible data loss**. Only for a real outage when the primary is unreachable.
- **Geo-restore drill**: use a **copy** of production. Simulate the outage (for example rename the source database), geo-restore to another server, and repoint the app.
- **ASR drill**: run a **test failover** of a recovery plan into an isolated test network. Production keeps running.
- Run drills **regularly**. Many industry standards require them.

**Example:** An insurance firm runs a DR drill every quarter. It does a planned failover of its failover group, checks the app, notes RTO = 4 minutes, then fails back.

## Test local HA in SQL Database and SQL MI
A **user-initiated failover** moves the database to another replica so you can test the app's **retry logic**.

| Target | PowerShell | Azure CLI |
|---|---|---|
| Single database | `Invoke-AzSqlDatabaseFailover` | No `az sql db failover`. Use `az rest` |
| Elastic pool | `Invoke-AzSqlElasticPoolFailover` | `az rest` |
| Managed Instance | `Invoke-AzSqlInstanceFailover` (`-ReadableSecondary` for BC) | `az sql mi failover` (`--replica-type ReadableSecondary`) |

```powershell
Invoke-AzSqlDatabaseFailover -ResourceGroupName rg-prod -ServerName sql-prod-weu -DatabaseName SalesDb
Invoke-AzSqlInstanceFailover -ResourceGroupName rg-prod -Name mi-prod-weu
```

- Limit: **one failover every 15 minutes** per database, elastic pool or managed instance.
- Portal: **Settings → Maintenance → Restart** (preview) uses the same API. Restarting a **pool restarts all its databases**.
- Not available for **Hyperscale readable secondary** replicas.
- MI failover is **blocked** until a new database's first full backup finishes, and while a **restore** is running.
- MI permission: **SQL Managed Instance Contributor** (or `Microsoft.Sql/managedInstances/failover/action`).

## Test geo DR: geo-replication and failover groups
Run these against the **secondary**.

| Feature | Planned (no data loss) | Forced (possible data loss) |
|---|---|---|
| Geo-replication, PowerShell | `Set-AzSqlDatabaseSecondary ... -Failover` | add `-AllowDataLoss` |
| Geo-replication, CLI | `az sql db replica set-primary` | add `--allow-data-loss` |
| Geo-replication, T-SQL (master of secondary server) | `ALTER DATABASE [SalesDb] FAILOVER;` | `ALTER DATABASE [SalesDb] FORCE_FAILOVER_ALLOW_DATA_LOSS;` |
| Failover group (SQL DB), PowerShell | `Switch-AzSqlDatabaseFailoverGroup` | add `-AllowDataLoss` |
| Failover group (SQL DB), CLI | `az sql failover-group set-primary` | add `--allow-data-loss` |
| Failover group (SQL MI) | `Switch-AzSqlDatabaseInstanceFailoverGroup` / `az sql instance-failover-group set-primary` | add allow-data-loss option |

```powershell
# -ServerName must be the SECONDARY server
Switch-AzSqlDatabaseFailoverGroup -ResourceGroupName rg-dr -ServerName sql-prod-neu -FailoverGroupName fog-sales
```

- `-TryPlannedBeforeForcedFailover` tries planned first, then forced.
- If the database is **in a failover group**, you can't use the geo-replication failover command. Fail over the **group**.
- Check roles with `Get-AzSqlDatabaseFailoverGroup` (`ReplicationRole`) or `az sql failover-group show`.
- The **Activity log** shows "Failover Azure SQL failover group". If Microsoft started it, **Event initiated by** shows `-`.
- **MI drills:** use a **planned** failover. MI groups can't fail over while **Seeding**. A new full backup runs after every failover, so failing back too soon can be delayed.

## Test HA/DR on SQL Server VMs
```sql
-- Planned: run on the TARGET secondary (sync commit + SYNCHRONIZED)
ALTER AVAILABILITY GROUP [AG1] FAILOVER;
-- Forced (DR only): run on a secondary
ALTER AVAILABILITY GROUP [AG1] FORCE_FAILOVER_ALLOW_DATA_LOSS;
-- After a forced failover, secondary DBs are SUSPENDED: resume on each secondary
ALTER DATABASE [SalesDb] SET HADR RESUME;
```

- PowerShell: `Switch-SqlAvailabilityGroup` (add `-AllowDataLoss` to force). FCI: `Move-ClusterGroup` or Failover Cluster Manager → Roles → **Move**.
- Check readiness: `is_failover_ready = 1` in `sys.dm_hadr_database_replica_cluster_states`, or **Failover Readiness** in the **Always On dashboard**.
- Before forcing, pick the secondary with the **highest `end_of_log_lsn`** (least data loss). **Block clients** from the old primary to avoid **split brain**.
- To fail over to an **async** replica with no data loss: switch it to **SYNCHRONOUS_COMMIT**, wait for **SYNCHRONIZED**, then run `FAILOVER`.

### Test your backups
A backup is only proven by a restore.
```sql
RESTORE VERIFYONLY FROM URL = 'https://stbackups.blob.core.windows.net/sql/SalesDb_full.bak' WITH CHECKSUM;
RESTORE DATABASE SalesDb_Test FROM URL = '...' WITH MOVE ..., STATS = 10;  -- on a test server
DBCC CHECKDB ('SalesDb_Test') WITH NO_INFOMSGS;                            -- the real proof
```
`RESTORE VERIFYONLY` checks the file is readable. It does **not** prove the database is consistent.

## Monitor: what to watch
**Azure level**
- **Azure Status page**: global outages (RSS feed).
- **Azure Service Health**: problems, maintenance and outages for your subscription. Alerts by email or SMS.
- **Resource Health**: health of one database or VM, and **why a failover happened** (planned maintenance or failure).
- **Activity log**: restores, database creation, failover operations.
- Status from CLI/PowerShell: `az sql db list`, `az sql mi list`, `Get-AzSqlDatabase`.

**Replication DMVs** (DMV = dynamic management view, a system view showing live server state)
| Platform | DMV | Key columns |
|---|---|---|
| AG on VM | `sys.dm_hadr_database_replica_states` | `synchronization_state_desc`, `synchronization_health_desc`, `is_suspended`, `log_send_queue_size`, `redo_queue_size`, `secondary_lag_seconds` |
| AG on VM | `sys.dm_hadr_availability_replica_states` | `role_desc`, `connected_state_desc` |
| Geo-replication / failover group | `sys.dm_geo_replication_link_status` (run in the **user database** on the primary; empty in master) | `replication_state_desc`, `replication_lag_sec`, `last_replication` |
| SQL DB / MI replicas | `sys.dm_database_replica_states` | `redo_queue_size`, `secondary_lag_seconds` |
| Log shipping | `sp_help_log_shipping_monitor`, `msdb.dbo.log_shipping_monitor_*` | last backup, copy and restore file and time |

- Big **log send queue** or `replication_lag_sec` → data at risk (**RPO**).
- Big **redo queue** → slower failover (**RTO**).

**Backup monitoring**
- SQL Database: `sys.dm_database_backups` (empty on Hyperscale). SQL MI: `msdb.dbo.backupset`, or Extended Events (XEvents).
- LTR backups: portal, `az sql db ltr-backup list`, `Get-AzSqlDatabaseLongTermRetentionBackup`.

**Example:** A retail app alerts when `replication_lag_sec` goes above 5 seconds, because its RPO is 5 seconds.

Design apps for availability too: keep the **app and database in the same region**, and add **retry logic**.

## Troubleshoot AGs on Azure VMs
| Symptom | Likely cause | Fix |
|---|---|---|
| Listener unreachable after failover (single subnet, VNN) | Load balancer probe not answered | Set `ProbePort` on the listener IP, enable **Floating IP**, open the probe port (e.g. 59999) in the firewall |
| Slow reconnect after failover | Client caches the old IP | `MultiSubnetFailover=True`. Else `RegisterAllProvidersIP=0`, `HostRecordTTL=300` |
| DNN listener connection fails | Driver lacks `MultiSubnetFailover` or wrong port | Use a supporting driver and the **DNN port** in the connection string |
| Errors **19407 / 19419** (lease expired), event **1135** (node removed) | Cluster monitoring too strict for Azure | **Relax** thresholds and timeouts (below) |
| Errors **35201 / 35206** | Endpoint port, NSG or firewall, or session timeout too low | Check port **5022**, NSGs, endpoint. Raise `SESSION_TIMEOUT` |
| Database **NOT SYNCHRONIZING / suspended** | Forced failover, disk/redo error, or user suspended it | Fix the cause, then `ALTER DATABASE ... SET HADR RESUME` |
| Cluster offline, **quorum lost** | Lost votes or witness | **Cloud witness**, odd number of votes. Emergency: `Start-ClusterNode -FixQuorum`, then force failover |
| "Not failing over group" | **Max failures in specified period** reached | Raise from 2 to **6** |

Relaxed settings for Azure:
```powershell
(Get-Cluster).SameSubnetThreshold = 40
(Get-Cluster).CrossSubnetThreshold = 40
Get-ClusterLog -Destination C:\Temp -TimeSpan 60 -UseLocalTime   # collect the cluster log
```
```sql
ALTER AVAILABILITY GROUP [AG1] SET (HEALTH_CHECK_TIMEOUT = 60000);  -- default 30000 ms
ALTER AVAILABILITY GROUP [AG1] SET (FAILURE_CONDITION_LEVEL = 2);   -- default 3
ALTER AVAILABILITY GROUP [AG1] MODIFY REPLICA ON 'SQLVM1' WITH (SESSION_TIMEOUT = 20); -- default 10 s
```
- **Lease timeout**: default 20 s, raise to **40 s** (in Failover Cluster Manager).
- Where to look: SQL error log, **AlwaysOn_health** Extended Events session (created by the wizard, start it yourself if you used T-SQL), **Always On dashboard**, **cluster log**, Azure **Activity log** and **Resource Health**.

**Example:** A 2-node AG keeps failing over at night with error 19407. The DBA raises `SameSubnetThreshold` to 40 and the lease timeout to 40 s. The false failovers stop.

## Troubleshoot geo-replication and failover groups
- **Lag keeps growing** (`replication_lag_sec` up, `redo_queue_size` rising): usually the **secondary is smaller** than the primary. Wait type `HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO`. Fix: **same tier and compute size**. Other causes: big index rebuilds, bulk loads, long queries on the readable secondary.
- `replication_lag_sec` is **NULL** for a long time → primary can't reach the secondary (network). Alert on it.
- **Stuck in SEEDING**: you can't fail over or connect yet. Seeding runs at about **500 GB/h (SQL DB)** and **360 GB/h (MI)**. MI group creation is cancelled after **6 days**.
- **MI roles don't switch**: check **VNet peering** and NSG/firewall rules. Both instances must share the **DNS zone**.
- **Logins fail after failover**: use **contained users** and **database-level firewall rules** (they replicate). For server logins, create them on the secondary with the **same SID**. MI system databases are not replicated.
- **Reconnect**: DNS TTL is **30 s**. Existing connections drop. Apps need **retry logic**.
- **Protect a critical transaction**: call `sp_wait_for_database_copy_sync` after commit. It waits until the log reaches the geo-secondary.

## Troubleshoot log shipping and backups
```sql
EXEC master.dbo.sp_help_log_shipping_monitor;
SELECT * FROM msdb.dbo.log_shipping_monitor_error_detail ORDER BY log_time DESC;
```
- Alert **14420** = no **backup** within the threshold. Alert **14421** = no **restore** within the threshold. The alert job runs on the **monitor server** (or primary/secondary if none).
- **Broken log chain** (restore error **4305**, "too recent to apply"): someone took an extra log backup outside the log shipping job, or switched to SIMPLE. Fix: restore the missing log, or re-seed from a full backup. Use **COPY_ONLY** for ad-hoc backups.
- Geo-secondaries: **PITR retention is not replicated** (it defaults to 7 days). Set it yourself.

## Remember for the exam
- Drills: **planned** failover (no data loss) in production and for failback. **Forced** only for real outages.
- Test app retry logic on SQL DB → `Invoke-AzSqlDatabaseFailover`. **One per 15 minutes**.
- Fail over a failover group → `Switch-AzSqlDatabaseFailoverGroup` on the **secondary** server.
- Primary region down → `-AllowDataLoss` / `--allow-data-loss` / `FORCE_FAILOVER_ALLOW_DATA_LOSS`.
- After AG forced failover → secondaries **suspended** → `SET HADR RESUME`.
- Planned AG failover needs **sync commit + SYNCHRONIZED**, run on the **target**.
- RPO metric: `replication_lag_sec`, `log_send_queue_size`. RTO metric: `redo_queue_size`.
- `sys.dm_geo_replication_link_status` → run in the **user database**, not master.
- Error 19407 on Azure VMs → relax **SameSubnetThreshold 40**, lease **40 s**, health check **60 s**.
- Log shipping alerts: **14420 backup**, **14421 restore**.

## Quick check
1. You want to check that an app reconnects correctly when its Azure SQL database fails over inside the region. Which command?
   <details><summary>Answer</summary>Invoke-AzSqlDatabaseFailover. It triggers a local HA failover. Failover group commands test regional DR instead.</details>
2. After a forced AG failover, the secondary databases show NOT SYNCHRONIZING. What do you run?
   <details><summary>Answer</summary>ALTER DATABASE ... SET HADR RESUME on each secondary. Forced failover suspends data movement.</details>
3. A geo-secondary keeps falling behind and shows wait type HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO. What is the fix?
   <details><summary>Answer</summary>Scale the secondary to the same service tier and compute size as the primary.</details>
