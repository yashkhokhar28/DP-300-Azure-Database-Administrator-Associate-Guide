# Troubleshoot a migration

> **Exam skills:** Troubleshoot a migration
> **Reading time:** ~9 min

## The big idea
Most migration failures come from four things: **network** (can't reach the source), **permissions** (storage or login rights), **backup format** (wrong kind of backup file) and **unsupported features** on the target. Know where to look, and know the common error messages.

Analogy: a parcel that never arrived. Was the address wrong (network)? Did nobody sign for it (permissions)? Was the box the wrong shape (format)? Or is the item banned at the destination (feature)?

## Where to look
| Place | What it shows |
|-------|---------------|
| **DMS → Monitor migrations** | Per database and per table status and error text |
| **Activity log** (resource → Activity log → Summary/JSON) | Resource-level failures, exact error messages |
| **Arc → Monitor and cutover** | LRS files detected, queued, skipped or unrestorable |
| LRS status | `Get-AzSqlInstanceDatabaseLogReplay` / `az sql midb log-replay show` |
| Failed MI operation | `Get-AzSqlInstanceOperation` / `az sql mi op show` |
| MI link state | `Get-AzSqlInstanceLink` / `az sql mi link show` / DMVs |
| SQL Server error log, XEvent profiler | Engine errors (link, certificates, seeding) |

## Connectivity problems
| Symptom | Cause | Fix |
|---------|-------|-----|
| "Error 26 – Error Locating Server/Instance" | **Named instance** or dynamic port | Open **UDP 1434** (SQL Browser) **and** the instance TCP port |
| DMS can't reach source, stuck | **SHIR** can't see the source or backups | Install SHIR on a machine with line of sight to the source; size it well |
| Can't read backups on an SMB share | Share rights | Give the migration account **and** the SQL service account full control. Test with `RESTORE HEADERONLY` |
| Timeout over ExpressRoute (DMS classic) | Missing service endpoints | Enable Service Bus, Storage and target database endpoints on the DMS subnet |
| MI link won't create | No private path, ports blocked | VPN/ExpressRoute, port **5022** both ways, MI HADR ports, NSG rules |

**Example:** A DBA migrates `SQL01\SALES` with DMS. It fails with error 26. The firewall only allows TCP 1433. Opening UDP 1434 and the instance's port fixes it.

## Backup rules that break migrations
For DMS (to MI or VM) and LRS:
- **One backup per file.** Backups **appended** into one file are not supported.
- Use **`WITH CHECKSUM`**. Without it you might migrate a corrupt database, and restores are slower.
- **One log file** per database. Databases with several log files are not supported.
- Only **.bak, .diff, .log** files. **No BACPAC/DACPAC** with LRS.

## LRS checklist
| Problem | Cause / fix |
|---------|-------------|
| LRS won't start | **SAS must be Read + List only.** Write makes it fail. Make it for the whole container |
| SAS "invalid" | Time window wrong (time zones). Regenerate with a wider window |
| Name conflict | A database with the **same name** already exists on MI |
| Files not found | Each database needs its **own flat folder**. Path can't contain the word **backup/backups** |
| Can't add new log backups | You used **autocomplete**. Use **continuous** mode |
| Job disappeared | Hit the **30-day** limit |
| Restarted from zero | MI failover during the first full restore or during cutover |
| "Unable to list directories" (portal) | **Your user** lacks **Storage Blob Data Reader** |
| **Error 2009** (managed identity not set up properly) | The **MI's managed identity** lacks **Storage Blob Data Reader** |
| "No results were found in directory" | No full backup uploaded yet |

Two more MI facts:
- MI holds back non-critical updates for up to **36 hours** during a migration. After that, an interrupted migration **restarts from a full restore**.
- **Business Critical cutover is slow** because it seeds 3 replicas. Fix: migrate to **General Purpose** then upgrade online, or use **MI link**.

## MI link problems
```sql
SELECT rs.synchronization_health_desc   -- HEALTHY / NOT_HEALTHY
FROM sys.availability_groups ag
JOIN sys.dm_hadr_availability_replica_states rs ON ag.group_id = rs.group_id
WHERE rs.is_local = 0 AND rs.role = 2 AND ag.is_distributed = 1 AND ag.name = '<link name>';
```
- Seeding progress: `sys.dm_hadr_physical_seeding_stats`, `sys.dm_hadr_automatic_seeding`.
- **First seeding is not resilient.** A network drop, restart or failover restarts it. Link creation auto-cancels after **6 days**.

| Error | Meaning | Fix |
|-------|---------|-----|
| 41962 | Link not started within 5 minutes | Check network, retry |
| 41973 / 41974 | Endpoint certificate not imported (SQL Server → MI / MI → SQL Server) | Re-import the certificate |
| 41976 / 41986 | AG not responding / connection failed | Check names, config, network |
| **1408 / 1412** | **Log truncated before seeding finished** (a log backup ran) | **Drop and recreate** the link. Pause log backups or use **trace flag 12381** |

Other link gotchas:
- Planned failover **times out** → the secondary is lagging. Wait for it.
- Both sides PRIMARY after forced failover (**split brain**) → back up the old primary, set it to SECONDARY, fail over properly.
- **Expired certificate** → rotate it.
- Don't drop the link before MI's **first full backup** after failover.
- Firewalls must allow **ICMP** and the SQL Server UCS protocol.
- A link made by hand in SSMS blocks creating one from the Arc portal.

## Azure SQL Database (DMS) problems
| You see | Why |
|---------|-----|
| Can't click Next | No tables on target. Tick **Migrate missing schema** |
| Computed columns missing | Not migrated |
| NULLs became default values | Columns with defaults get the default |
| Some tables not migrated | **Empty tables are skipped** |
| Table times out | **Large BLOB** columns |
| Table name rejected | **Double-byte** characters. Rename, migrate, rename back |
| Database name rejected | Reserved words or semicolons |
| CDC won't enable on target | Disable CDC on source before schema migration |
| Slow with thousands of tables | ADF overhead per table |

Permissions: source **db_datareader + VIEW ANY DEFINITION**; target **db_owner**. Schema migration also needs db_owner on the source and SHIR 5.37+.

Old DMS (classic) errors you may still see in questions: max **4 databases** per activity; an activity stuck in **Queued** means the instance is full; you can't stop DMS while activities run.

## After the migration
| Problem | Fix |
|---------|-----|
| **Orphaned users** (user without matching login) | `ALTER USER [app] WITH LOGIN = [app];` |
| Logins, Agent jobs, linked servers missing | Database-level tools don't move them. Script them |
| FILESTREAM/FileTable fails on MI | Not supported. Use a **VM** |
| In-Memory OLTP fails on MI | Needs **Business Critical** |
| Service Broker unusable on MI | It was off on the source. Enable it and re-migrate |
| Can't enable ADR on MI | ADR was off on the source (SQL 2019+). Enable and re-migrate |
| Queries slower | Update statistics, check **compatibility level**, compare plans with **Query Store** |

**Example:** After restoring to a VM, the app can't log in. The database user `app` exists, but its SID doesn't match the new server login. `ALTER USER [app] WITH LOGIN = [app]` fixes it.

## Remember for the exam
- **Error 2009** → give the **MI managed identity** Storage Blob Data Reader.
- LRS won't start, SAS has Write → SAS must be **Read + List only**.
- **Error 1412** on MI link → log truncated during seeding → drop and recreate, pause log backups or **TF 12381**.
- Named instance unreachable → **UDP 1434** + instance port.
- DMS to SQL Database is **offline only**. Empty tables are skipped. Computed columns aren't migrated.
- Slow Business Critical cutover → migrate to GP then upgrade, or use MI link.
- Orphaned users → `ALTER USER ... WITH LOGIN`.

## Quick check
1. LRS fails with error 2009. What do you fix?
   <details><summary>Answer</summary>Grant the managed instance's managed identity the Storage Blob Data Reader role on the storage account.</details>
2. MI link creation fails with error 1412. What happened?
   <details><summary>Answer</summary>A log backup truncated the log before seeding finished. Recreate the link and pause log backups (or use TF 12381).</details>
3. After DMS to SQL Database, three tables are missing. They had no rows. Is this a failure?
   <details><summary>Answer</summary>No. DMS skips empty tables by design.</details>
