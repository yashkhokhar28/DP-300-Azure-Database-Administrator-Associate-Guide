## 🧪 **Why Test and Troubleshoot HA/DR?**

An HA/DR design is only a **theory** until you have failed it over, measured it, and failed it back.
DP-300 expects you to **plan a testing procedure**, **monitor** the solution, and **troubleshoot** it when replicas fall behind or failovers fail.

- **DR drill** = simulate the outage → recover → **validate the app still works** (connection strings, logins, basic functions).
- Measure the **actual RTO** (time to be back online) and **actual RPO** (data lost) and compare them with the targets.
- Microsoft recommends running drills **regularly**. Most industry standards require them for business continuity certification.

---

## 📋 **Plan a Testing Procedure**

| Step | What to do |
| --- | --- |
| 1. Scope | List every component: DB, app tier, DNS, storage, logins, jobs, firewall/NSG rules |
| 2. Baseline | Record current lag (`replication_lag_sec`, `log_send_queue_size`) and target RPO/RTO |
| 3. Choose test type | **Planned failover** (no data loss, safe in production) or **forced** (only in a test environment) |
| 4. Execute | Use the commands below. Note start/end timestamps |
| 5. Validate | App connects through the **listener**, logins work, jobs run, reads/writes succeed |
| 6. Record | Actual RTO, actual RPO, issues found, runbook fixes |
| 7. Failback | Use a **planned** failover to return to the original region/replica |

- **Planned failover** = full sync first → **no data loss** → use it for **DR drills in production**, relocation, and **failback**.
- **Forced failover** = switch immediately → **possible data loss** → only for a real outage when the primary is unreachable.
- **Geo-restore drill**: do it on a **copy** of production. Simulate the outage (e.g., rename the source DB), geo-restore to another server, and repoint the app.

---

## ☁️ **Test HA in Azure SQL Database / Managed Instance (Local Failover)**

A **user-initiated failover** restarts the engine on another replica/node so you can test **retry logic** and the impact on open sessions.

| Target | PowerShell | Azure CLI |
| --- | --- | --- |
| Single database | `Invoke-AzSqlDatabaseFailover` | No `az sql db failover` command → use `az rest` (REST "Databases - Failover") |
| Elastic pool | `Invoke-AzSqlElasticPoolFailover` | `az rest` (REST "Elastic Pools - Failover") |
| Managed Instance | `Invoke-AzSqlInstanceFailover` (`-ReadableSecondary` for BC) | `az sql mi failover` (`--replica-type ReadableSecondary`) |

```powershell
# Azure SQL Database - restart/failover to test application resiliency
Invoke-AzSqlDatabaseFailover -ResourceGroupName "rg-prod" -ServerName "sql-prod-weu" -DatabaseName "SalesDb"

# Managed Instance - fail over the primary (GP and BC), then the readable secondary (BC only)
Invoke-AzSqlInstanceFailover -ResourceGroupName "rg-prod" -Name "mi-prod-weu"
Invoke-AzSqlInstanceFailover -ResourceGroupName "rg-prod" -Name "mi-prod-weu" -ReadableSecondary
```

- **Throttling**: only **one failover every 15 minutes** per **database or elastic pool**, and per **managed instance**.
- The portal has **Settings → Maintenance → Restart** (preview) for databases and pools. It uses the same API, and restarting a **pool restarts all its databases**.
- The Failover command is **not available for Hyperscale readable secondary replicas**.
- MI failover is **blocked** until the first full backup of a new DB completes, and while a **restore is in progress**.
- MI permission: **SQL Managed Instance Contributor** (or `Microsoft.Sql/managedInstances/failover/action`).
- Check progress on MI: GP → `sys.dm_os_sys_info.sqlserver_start_time` changes. BC → `sys.dm_hadr_fabric_replica_states` shows a new primary.

---

## 🌍 **Test DR: Active Geo-Replication and Failover Groups**

### Active geo-replication (per database, Azure SQL Database only)

| Method (run against the **secondary**) | Planned (no data loss) | Forced (possible data loss) |
| --- | --- | --- |
| PowerShell | `Set-AzSqlDatabaseSecondary ... -PartnerResourceGroupName rg-prod -Failover` | add `-AllowDataLoss` |
| Azure CLI | `az sql db replica set-primary -g rg-dr -s sql-prod-neu -n SalesDb` | add `--allow-data-loss` |
| T-SQL (in `master` of the secondary server) | `ALTER DATABASE [SalesDb] FAILOVER;` | `ALTER DATABASE [SalesDb] FORCE_FAILOVER_ALLOW_DATA_LOSS;` |

- After geo-replication failover, the **endpoint changes** (new logical server) → the app must update its connection string.
- If the DB is **in a failover group**, you **can't** use the geo-replication failover command. Fail over the **group** instead.

### Failover groups (SQL Database and SQL Managed Instance)

```powershell
# SQL Database: -ServerName must be the SECONDARY server
Switch-AzSqlDatabaseFailoverGroup -ResourceGroupName "rg-dr" -ServerName "sql-prod-neu" -FailoverGroupName "fog-sales"
# Forced: add -AllowDataLoss   |  Try planned first, then forced: -TryPlannedBeforeForcedFailover

# SQL Managed Instance: pipe the group from the SECONDARY location
Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName "rg-dr" -Location "northeurope" -Name "fog-mi" |
    Switch-AzSqlDatabaseInstanceFailoverGroup
```

- CLI: `az sql failover-group set-primary --name fog-sales -g rg-dr --server sql-prod-neu` (add `--allow-data-loss` to force). For MI, use `az sql instance-failover-group set-primary`.
- Check roles before/after with `Get-AzSqlDatabaseFailoverGroup` (`ReplicationRole`) or `az sql failover-group show`.
- Track the operation in the **Activity log**: "Failover Azure SQL failover group". If Microsoft triggered it, **Event initiated by** shows `-`.
- **MI drills**: Microsoft recommends a **planned** failover. Forced failover without data loss is only possible if you stop the workload, close all connections, and see status **Synchronizing** on **both** instances.
- MI failover groups can't fail over while in the **Seeding** state. A new full backup runs **after every failover**, so failing back too soon can be delayed.

### ⚙️ **Failover policy: Customer managed vs Microsoft managed**

| Policy | CLI/PowerShell value | Behaviour |
| --- | --- | --- |
| **Customer managed (recommended)** | `Manual` | You decide when to fail over, including selective/forced failover |
| **Microsoft managed** | `Automatic` | Microsoft fails over **all** Microsoft-managed groups in the region, only for a wide regional outage, after the **grace period** (minimum **1 hour**) |

- Microsoft's docs now say to **use customer managed to test and run your DR plan**, and **not to rely on** Microsoft-managed failover.
- Microsoft-managed failover is **not** done for single groups. Failback is always **manual**.
- `--failover-policy` in `az sql failover-group create` now **defaults to Manual**. `--grace-period` / `GracePeriodWithDataLossHours` defaults to **1**.

---

## 🖥️ **Test HA/DR on SQL Server in Azure VMs**

```sql
-- Planned manual failover: run on the TARGET secondary (sync-commit + SYNCHRONIZED)
ALTER AVAILABILITY GROUP [AG1] FAILOVER;

-- Forced failover (DR only): run on a SECONDARY/RESOLVING replica
ALTER AVAILABILITY GROUP [AG1] FORCE_FAILOVER_ALLOW_DATA_LOSS;

-- After a forced failover, ALL secondary DBs are SUSPENDED: resume each one on each secondary
ALTER DATABASE [SalesDb] SET HADR RESUME;
```

```powershell
Switch-SqlAvailabilityGroup -Path SQLSERVER:\Sql\SQLVM2\DEFAULT\AvailabilityGroups\AG1                 # planned
Switch-SqlAvailabilityGroup -Path SQLSERVER:\Sql\SQLVM3\DEFAULT\AvailabilityGroups\AG1 -AllowDataLoss  # forced
Move-ClusterGroup -Name "SQL Server (MSSQLSERVER)" -Node "SQLVM2"   # FCI: move the role to another node
```

- Check readiness first: `is_failover_ready = 1` in `sys.dm_hadr_database_replica_cluster_states`, or **Failover Readiness** in the **Always On dashboard**.
- Before you force a failover, pick the secondary with the **highest `end_of_log_lsn`** (least data loss). Also **block clients** from the old primary to avoid **split brain**.
- To fail over to an **async DR replica without data loss**, temporarily switch it to **SYNCHRONOUS_COMMIT**, wait for **SYNCHRONIZED**, then run `FAILOVER`.
- **FCI test**: Failover Cluster Manager → Roles → **Move → Best Possible Node / Select Node**. Then confirm clients reconnect through the **virtual network name or DNN**.
- **Test restores** (backups are only proven by a restore):

```sql
RESTORE VERIFYONLY FROM URL = 'https://stbackups.blob.core.windows.net/sql/SalesDb_full.bak'
WITH CHECKSUM;            -- checks the backup is complete/readable (+ page checksums if taken WITH CHECKSUM)

RESTORE DATABASE SalesDb_Test FROM URL = '...SalesDb_full.bak' WITH MOVE ..., STATS = 10;  -- test server
DBCC CHECKDB ('SalesDb_Test') WITH NO_INFOMSGS;   -- the real proof of a usable backup
```

---

## 📈 **Monitor Refresher (What to Watch)**

| Platform | Key DMV / tool | Key columns |
| --- | --- | --- |
| AG on VM | `sys.dm_hadr_database_replica_states` | `synchronization_state_desc`, `synchronization_health_desc`, `is_suspended`, `suspend_reason_desc`, `log_send_queue_size` (KB), `redo_queue_size` (KB), `secondary_lag_seconds`, `last_commit_time` |
| AG on VM | `sys.dm_hadr_availability_replica_states` | `role_desc`, `connected_state_desc`, `synchronization_health_desc` |
| Geo-replication / FOG | `sys.dm_geo_replication_link_status` (run on the **primary** DB, not master) | `replication_state_desc` (SEEDING, CATCH_UP, SUSPENDED), `replication_lag_sec`, `last_replication` |
| SQL DB secondary redo | `sys.dm_database_replica_states` | `redo_queue_size`, `secondary_lag_seconds` |
| Log shipping | `sp_help_log_shipping_monitor`, msdb `log_shipping_monitor_*` tables | last backup/copy/restore file and time |

```sql
-- AG health at a glance (run on the primary)
SELECT ar.replica_server_name, DB_NAME(drs.database_id) AS db,
       drs.synchronization_state_desc, drs.synchronization_health_desc,
       drs.is_suspended, drs.suspend_reason_desc,
       drs.log_send_queue_size AS send_queue_kb, drs.redo_queue_size AS redo_queue_kb,
       drs.secondary_lag_seconds
FROM sys.dm_hadr_database_replica_states drs
JOIN sys.availability_replicas ar ON ar.replica_id = drs.replica_id;
```

- Rough rule: a large **log send queue** → data at risk (**RPO**). A large **redo queue** → slower failover (**RTO**).

---

## 🔧 **Troubleshoot Availability Groups on Azure VMs**

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| App can't reach the **listener** after failover (single subnet, VNN) | LB probe not answered, Floating IP off, probe port not set on the cluster IP resource | Set `ProbePort` on the listener IP resource, enable **Floating IP**, open the probe port (e.g., **59999**) in the firewall |
| Slow reconnect after failover | Client caches the old IP (default 20 min) | `MultiSubnetFailover=True`. If the client can't use it: `RegisterAllProvidersIP=0`, `HostRecordTTL=300` |
| DNN listener connection fails | Driver lacks `MultiSubnetFailover`, or the wrong port is used | Use a driver that supports `MultiSubnetFailover=True` and put the **unique DNN port** in the connection string |
| Event **1069 status 10048** | Probe port in the dynamic range is taken by another process | `netsh int ipv4 add excludedportrange tcp startport=59999 numberofports=1 store=persistent` |
| Error **19407 / 19419** (lease expired) | Monitoring too aggressive for Azure | Relax the cluster thresholds and lease timeout (see below) |
| Error **35201 / 35206** | Endpoint, NSG, or firewall issue, or session timeout too low | Check port **5022**, NSGs, and the endpoint state. Raise `SESSION_TIMEOUT` |
| Event **1135** (node removed) | Missed heartbeats | `SameSubnetThreshold`/`CrossSubnetThreshold` = **40** |
| DB **NOT SYNCHRONIZING / suspended** | Forced failover (`SUSPEND_FROM_PARTNER`), redo/disk error, or a user suspended it | Fix the cause (disk space, file path), then `ALTER DATABASE ... SET HADR RESUME` |
| Cluster offline / **quorum lost** | Votes lost, witness unreachable | Use a **cloud witness**, keep an odd number of votes (no votes for DR nodes). In an emergency, force quorum (`Start-ClusterNode -FixQuorum`) and then **force failover** |
| "Not failing over group..." | **Max failures in specified period** reached | Raise it from 2 to **6** |

```powershell
# Relaxed Azure settings (apply immediately, no restart)
(Get-Cluster).SameSubnetThreshold = 40
(Get-Cluster).CrossSubnetThreshold = 40

# VNN + load balancer: tie the listener IP resource to the LB probe
Get-ClusterResource "AG1_10.0.0.20" | Set-ClusterParameter -Multiple @{
  "Address"="10.0.0.20";"ProbePort"=59999;"SubnetMask"="255.255.255.255";"Network"="Cluster Network 1";"EnableDhcp"=0}

# Collect the cluster log for root-cause analysis (last 60 minutes)
Get-ClusterLog -Destination C:\Temp -TimeSpan 60 -UseLocalTime
```

```sql
ALTER AVAILABILITY GROUP [AG1] SET (HEALTH_CHECK_TIMEOUT = 60000);   -- default 30000 ms
ALTER AVAILABILITY GROUP [AG1] SET (FAILURE_CONDITION_LEVEL = 2);    -- default 3
ALTER AVAILABILITY GROUP [AG1] MODIFY REPLICA ON 'SQLVM1' WITH (SESSION_TIMEOUT = 20);  -- default 10 s
-- Lease timeout (default 20 s → 40 s) is set in Failover Cluster Manager. Keep it < 2 x SameSubnetThreshold x SameSubnetDelay (max 80 s)
```

- **Diagnostic sources**: SQL error log, **AlwaysOn_health** XEvent session (created automatically by the **wizard**. Start it manually if you used T-SQL), **Always On dashboard**, **cluster log**, plus Azure **Activity log** and **Resource Health** (VM host events).
- **Multi-subnet** deployment removes the need for a load balancer or DNN and is now the **preferred** design. Prefer **DNN** over VNN+LB when you have a single subnet.

---

## 🌐 **Troubleshoot Geo-Replication and Failover Groups**

- **Growing lag**: `replication_lag_sec` rising, or `redo_queue_size` > 0 and growing on the secondary.
  - Usual cause: the **secondary is smaller** than the primary (wait type `HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO` = primary log rate is throttled). Fix: use the **same service tier and compute size**.
  - Other causes: large index rebuilds or bulk loads (SCH-M locks block redo), long transactions on the readable secondary.
- `replication_lag_sec` = **NULL** for a long time → the primary can't talk to the secondary (connectivity issue). Alert on it.
- **Stuck in SEEDING**: you can't fail over or connect yet. Seeding runs at about **500 GB/h (SQL DB)** and **360 GB/h (MI)**. MI failover group creation is **cancelled after 6 days**.
- **MI roles don't switch** → check **VNet peering** (global peering recommended) and NSG/firewall rules between subnets. Both instances must share the **DNS zone** (`DNSZonePartner`).
- **Logins fail after failover**: use **contained users** and **database-level firewall rules** (they replicate). For server logins, create them on the secondary with the **same SID**. MI system DBs (master, msdb: logins, Agent jobs) are **not replicated**.
- **Listeners**: read-write `<fog>.database.windows.net` and read-only `<fog>.secondary.database.windows.net` (MI: `<fog>.<zone_id>.database.windows.net`). DNS TTL is **30 s**, so clients must **reconnect**. Add `ApplicationIntent=ReadOnly` for read-only traffic.
- **App reconnect**: implement **retry logic** with backoff for transient errors. Existing connections are dropped and must be re-opened.
- **Protect critical transactions**: call `sp_wait_for_database_copy_sync` after commit (blocks until the log is hardened on the geo-secondary).

---

## 📦 **Troubleshoot Log Shipping and Backups**

```sql
EXEC master.dbo.sp_help_log_shipping_monitor;   -- status summary (monitor/primary/secondary)
SELECT * FROM msdb.dbo.log_shipping_monitor_error_detail ORDER BY log_time DESC;
SELECT * FROM msdb.dbo.log_shipping_monitor_secondary;  -- last_copied_file, last_restored_file, latency
```

- Alert errors: **14420** (no backup within the threshold) and **14421** (no restore within the threshold). The alert job runs on the **monitor server** (or on the primary/secondary if there's no monitor).
- **Broken LSN chain** (restore error **4305**, "too recent to apply"): someone took an ad-hoc log backup outside the LS job, or switched to SIMPLE recovery. Fix: restore the missing log, or re-seed from a full backup. Use **`COPY_ONLY`** for ad-hoc backups.
- **Azure SQL DB backups**: check `sys.dm_database_backups` (`backup_type` D/I/L, `in_retention`). It returns nothing on **Hyperscale** (snapshot backups). MI: `msdb.dbo.backupset`. For geo-secondaries, **PITR retention isn't replicated** (defaults to 7 days), so set it manually.
- **LTR**: list with `Get-AzSqlDatabaseLongTermRetentionBackup` / `az sql db ltr-backup list`. Restore with `Restore-AzSqlDatabase -FromLongTermRetentionBackup`. On MI failover groups, change the LTR policy on **both** instances.
---

## ✅ **Summary**

- Use a **planned (no data loss) failover** for drills and failback. Use **forced/AllowDataLoss** only for real outages or test copies.
- Local HA test: `Invoke-AzSqlDatabaseFailover`, `Invoke-AzSqlElasticPoolFailover`, `Invoke-AzSqlInstanceFailover` / `az sql mi failover`, limited to **one per 15 min**.
- Geo DR test: `Set-AzSqlDatabaseSecondary -Failover`, `Switch-AzSqlDatabaseFailoverGroup` (run on the **secondary**), `az sql failover-group set-primary`.
- The **customer managed** failover policy is the recommended choice. Microsoft managed is a last resort (all groups in the region, at least a 1 h grace period).
- Troubleshoot AGs with DMVs, the **AlwaysOn_health** session, the **cluster log**, and Azure-relaxed **threshold/lease** settings. Troubleshoot geo-replication with `sys.dm_geo_replication_link_status`.

---

## 🎯 **Exam Tips / Traps**

- "Test the app's retry logic against an **HA** failover of Azure SQL DB" → `Invoke-AzSqlDatabaseFailover` (**not** a failover group command).
- "Fail over the failover group with **no data loss**" → `Switch-AzSqlDatabaseFailoverGroup` **without** `-AllowDataLoss`, run against the **secondary** server.
- "The primary region is down" → forced failover: `-AllowDataLoss` / `--allow-data-loss` / `FORCE_FAILOVER_ALLOW_DATA_LOSS`.
- Geo-replication failover **changes the server name**. Failover groups keep the same **listener** names. Pick FOG when "no connection string changes".
- After an AG **forced failover**, secondaries are **SUSPENDED** → `ALTER DATABASE ... SET HADR RESUME`.
- Planned AG failover needs **synchronous-commit + SYNCHRONIZED** and must run **on the target secondary**.
- Frequent AG failovers on Azure VMs with error 19407 → **relax** `SameSubnetThreshold` (40), lease timeout (40 s), and health check timeout (60 s).
- Listener unreachable on a single-subnet VNN → check the **LB health probe / ProbePort / Floating IP**. Better: **DNN** or **multi-subnet**.
- RPO metric = `replication_lag_sec` / `log_send_queue_size`. RTO-related metric = `redo_queue_size`.
- `RESTORE VERIFYONLY` doesn't prove the database is consistent. Only a **test restore + DBCC CHECKDB** does.
- `sys.dm_geo_replication_link_status` returns an **empty set in master**. Query it in the user database.

> Source: https://learn.microsoft.com/en-us/azure/azure-sql/database/high-availability-sla-local-zone-redundancy , https://learn.microsoft.com/en-us/azure/azure-sql/database/restart-database , https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/user-initiated-failover , https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-sql-db , https://learn.microsoft.com/en-us/azure/azure-sql/database/failover-group-configure-sql-db , https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/failover-group-sql-mi , https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/failover-group-configure-sql-mi , https://learn.microsoft.com/en-us/azure/azure-sql/database/active-geo-replication-overview , https://learn.microsoft.com/en-us/azure/azure-sql/database/disaster-recovery-drills , https://learn.microsoft.com/en-us/azure/azure-sql/database/troubleshoot-geo-replication-redo , https://learn.microsoft.com/en-us/powershell/module/az.sql/switch-azsqldatabasefailovergroup , https://learn.microsoft.com/en-us/cli/azure/sql/failover-group , https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/hadr-cluster-best-practices , https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/availability-group-vnn-azure-load-balancer-configure , https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/perform-a-planned-manual-failover-of-an-availability-group-sql-server , https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server , https://learn.microsoft.com/en-us/sql/database-engine/availability-groups/windows/always-on-extended-events , https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-hadr-database-replica-states-transact-sql , https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-geo-replication-link-status-azure-sql-database , https://learn.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-database-backups-azure-sql-database , https://learn.microsoft.com/en-us/sql/database-engine/log-shipping/monitor-log-shipping-transact-sql
