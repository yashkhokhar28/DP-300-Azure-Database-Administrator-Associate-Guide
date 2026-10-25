### ✅ **Troubleshoot a Migration to Azure SQL — Explained**

---

## 🧭 **First: Which Migration Tools Still Exist? (Oct 2026)**

Several tools in the older course notes are **retired**. Exam questions can still mention them, so know what they did — but know the replacements too.

| Tool | Status | Replacement |
| --- | --- | --- |
| **Data Migration Assistant (DMA)** | ❌ **Retired 16 Jul 2025** | Azure Arc migration assessment, SSMS migration component (for version upgrades) |
| **Azure Data Studio + Azure SQL migration extension** | ❌ **Retired 28 Feb 2026** | Azure DMS portal / PowerShell / CLI, SQL Server migration in Azure Arc, SSMS migration component |
| **Azure Database Migration Service (classic)** – SQL Server scenarios | ❌ **Retired 15 Mar 2026** | Azure Database Migration Service (current) |
| **Azure Database Migration Service (DMS)** | ✅ Current | — |
| **SQL Server migration in Azure Arc** | ✅ Current (assess + migrate to MI via **LRS** or **MI link**) | — |
| **Log Replay Service (LRS)** / **Managed Instance link** | ✅ Current | — |

**Current tool by target:**

| Target | Portal | CLI |
| --- | --- | --- |
| **Assessment** | Azure Arc migration assessment / Azure Migrate | DMS PowerShell / CLI |
| **Azure SQL Managed Instance** | SQL Server migration in Azure Arc (LRS or MI link) | DMS PowerShell / CLI |
| **SQL Server on Azure VM** | Azure DMS portal (online) / SSMS migration component | DMS PowerShell / CLI |
| **Azure SQL Database** | Azure DMS portal (**offline + schema**) / Striim (online, third party) | DMS PowerShell / CLI |

---

## 🔎 **Where to Look When a Migration Fails**

| Place | What it tells you |
| --- | --- |
| **DMS → Monitor migrations** | Per-database & per-table status (Succeeded / Canceled / Error) and error details |
| **Azure Activity log** (resource → *Activity log* → event → **Summary / JSON** tabs) | Resource-level failures and exact error messages — copy these into support tickets |
| **Arc → Monitor and cutover** | Files **detected / queued / skipped / unrestorable** for LRS; jobs kept **28 days** |
| **LRS status** | `Get-AzSqlInstanceDatabaseLogReplay` / `az sql midb log-replay show` |
| **Failed MI operation details** | `Get-AzSqlInstanceOperation` / `az sql mi op show` |
| **MI link state** | `Get-AzSqlInstanceLink` / `az sql mi link show` / T-SQL DMVs (below) |
| **SQL Server error log / XE Profiler** | Engine-side errors (link, certificates, seeding) |

---

## 🌐 **Connectivity Problems (All Migration Types)**

| Symptom | Cause | Fix |
| --- | --- | --- |
| *"Error 26 – Error Locating Server/Instance Specified"* | Source is a **named instance** or uses a **dynamic port** | Allow **UDP 1434** (SQL Browser) **and** the instance's TCP port |
| DMS can't reach source / migration stuck | **Self-hosted integration runtime (SHIR)** can't see the source or backups | Install SHIR on a machine with line of sight to the source; it is also the **compute** for the copy, so size it properly |
| Can't read backups on an **SMB share** | Share permissions | Give **full control** to the Windows account used for migration **and** the SQL Server service account; test with `RESTORE HEADERONLY FROM DISK = N'\\share\full.bak'` |
| Timeout over **ExpressRoute** (DMS classic) | Missing service endpoints | Enable **Service Bus, Storage and target database** service endpoints on the DMS subnet |
| **MI link** won't create | No private path / ports blocked | Link needs **private** connectivity (VPN / ExpressRoute), port **5022** both ways, MI **HADR port range**, NSG rules — see MI link section |

---

## 📦 **Backup-Based Migrations (DMS to MI / VM, Log Replay Service)**

### ✅ Backup rules that cause most failures

- **One backup per file** — DMS doesn't support backups **appended** to a single file.
- Use **`WITH CHECKSUM`** — without it you can migrate a corrupt DB, and restores are slower.
- **Single transaction log file** — databases with multiple log files aren't supported.
- Only **`.bak`, `.diff`, `.log`** — **no BACPAC/DACPAC** with LRS.

```sql
BACKUP DATABASE Sales TO URL = 'https://acct.blob.core.windows.net/migration/Sales/Sales_full.bak'
WITH CHECKSUM, COMPRESSION;
BACKUP LOG Sales TO URL = 'https://acct.blob.core.windows.net/migration/Sales/Sales_log1.trn'
WITH CHECKSUM, COMPRESSION;
```

### 🔐 Log Replay Service (LRS) checklist

| Problem | Cause / Fix |
| --- | --- |
| LRS fails to start | **SAS token must be Read + List only** (adding Write makes it fail), generated for the **whole container**, not via a stored access policy, copied **after the `?`** |
| SAS token "invalid" | Validity window wrong (time-zone mismatch) → regenerate with a wider window |
| Database name conflict | A DB with the **same name already exists** on the MI → rename one |
| Files not found | Each DB needs its **own flat folder** (no nested folders); path/container can't contain the reserved word **`backup`/`backups`** |
| Can't add new log backups | You used **autocomplete** mode (needs the whole chain up front + last backup file name) → use **continuous** mode |
| Job disappeared | A single LRS job runs max **30 days**, then is canceled |
| Restarted from scratch | A **failover** during the first full restore, or after cutover starts, aborts the job → keep the last log backup small |
| *"Unable to list directories"* (portal) | **Your user** lacks **Storage Blob Data Reader** on the storage account |
| **Error 2009** – *managed identity is not set up properly* | The **MI's managed identity** lacks **Storage Blob Data Reader** (or the identity changed) |
| *"No results were found in directory"* | No full backup uploaded yet |

### ⏱️ Migration resets & slow cutovers (MI)

- MI holds **non-critical updates for up to 36 hours** during migration; after that (or for critical updates) an interrupted migration **resets to a full restore**.
- **Business Critical** cutover is slow: after cutover the DB is unavailable while it **seeds 3 secondary replicas** (hours for big DBs).
  - Workaround: migrate to **General Purpose**, then **upgrade to Business Critical** (online), **or** use **MI link**.

---

## 🔗 **Managed Instance Link Troubleshooting**

**Check health (run on the primary):**

```sql
DECLARE @link_name varchar(max) = '<DAGname>';
SELECT rs.synchronization_health_desc AS [Link sync health]   -- HEALTHY / NOT_HEALTHY
FROM sys.availability_groups ag
JOIN sys.dm_hadr_availability_replica_states rs ON ag.group_id = rs.group_id
WHERE rs.is_local = 0 AND rs.role = 2 AND ag.is_distributed = 1 AND ag.name = @link_name;
```

- Seeding progress: `sys.dm_hadr_physical_seeding_stats` + `sys.dm_hadr_automatic_seeding`.
- **Initial seeding isn't resilient** — a network drop, restart or failover **restarts seeding**; link creation is **auto-canceled after 6 days**.

| Error | Meaning | Fix |
| --- | --- | --- |
| **41962** | Link not initiated within 5 minutes | Check network, retry |
| **41973** | SQL Server endpoint certificate not imported into MI | Re-import cert |
| **41974** | MI endpoint certificate not imported into SQL Server | Re-import cert |
| **41976** | Availability group not responding | Check names/config |
| **41986** | Connection failed / secondary not responsive | Check names, config, network |
| **47521** | Secondary didn't receive the request | Check AG & DB health on primary |
| **41977** | Target database not responsive | Check link parameters |
| **1408 / 1412** | **Log truncated before seeding finished** (log backup ran during seeding) | **Drop & recreate** the link; prevent with **trace flag 12381** or pause log backups during seeding |

**Other link gotchas:**

- **Planned failover times out** → secondary is lagging; wait for it to catch up / pause workload.
- **Split-brain after forced failover** (both replicas PRIMARY) → back up the old primary (tail-log or copy-only), `ALTER AVAILABILITY GROUP [dag] SET (ROLE = SECONDARY)` on it, then fail over properly.
- **Expired certificate** → link fails; rotate the certificate.
- **Don't drop the link before MI's first full backup** completes after failover — otherwise the DB can become unavailable after a restart (check `msdb.dbo.backupset`).
- Firewalls must allow **ICMP** (else false negatives) and the **SQL Server UCS protocol** (else the test passes but the link fails).
- A link created manually (SSMS/T-SQL) **blocks** creating one from the Arc portal → drop link, certificates and endpoint first.

---

## 🗄️ **Azure SQL Database Target (DMS, Offline)**

- **Offline only** — DMS has **no online migration to Azure SQL Database**; downtime starts when the migration starts.
- Uses **Azure Data Factory pipelines** with the **SHIR as compute**.
- **Permissions:** source login needs **db_datareader + VIEW ANY DEFINITION**; target login needs **db_owner**.
  - **Schema migration** additionally needs **db_owner on the source**, SHIR **5.37+**, and the target login in **##MS_DatabaseManager##, ##MS_DatabaseConnector##, ##MS_DefinitionReader##, ##MS_LoginManager##**.
- If **no tables exist on the target**, you must tick **Migrate missing schema** (otherwise *Next* is disabled).

**Status flow:** Creating → Preparing for copy → Copying → Copy finished → Rebuilding indexes → Succeeded

**Known limitations that look like "bugs":**

| You see | Why |
| --- | --- |
| Computed columns missing | **Computed columns aren't migrated** |
| `NULL`s replaced by default values | Columns with **default constraints** get the default instead of NULL |
| Some tables "not migrated" | **0-row tables are skipped** |
| Table fails with timeout | **Large BLOB columns** can time out |
| Table name rejected | **Double-byte characters** in table names aren't supported → rename, migrate, rename back |
| DB name rejected | Reserved words or **semicolons** in DB name aren't supported |
| CDC won't enable on target | **Disable CDC on the source** before schema migration (else CDC objects are copied) |
| Slow with thousands of tables | ADF starts each table separately — overhead per table |

---

## 🔁 **Legacy DMS (Classic) Errors — Recognise in Older Questions**

- **Max 4 databases per migration activity** → split into more activities / another DMS instance.
- **Activity stuck in "Queued"** → instance at capacity; it starts automatically when others finish.
- **Can't stop DMS** → activities still running; stop/delete them first.

---

## 🩹 **Post-Migration Problems**

| Problem | Fix |
| --- | --- |
| Can't enable **Accelerated Database Recovery** on MI | ADR was **off on the source** (SQL 2019+) → enable on source and re-migrate |
| **Restore failures** later on MI | ADR **persistent version store not on PRIMARY** filegroup on source → fix and re-migrate |
| **Service Broker** unusable on MI | It was **disabled on the source** → enable before migrating |
| **Orphaned users** (VM / MI restore) | Re-map: `ALTER USER [app] WITH LOGIN = [app];` |
| Logins, Agent jobs, linked servers missing | Database-level migrations don't move **server-level objects** → script them separately |
| **FILESTREAM / FileTable** fail on MI | Not supported on MI → choose **SQL Server on Azure VM** |
| **In-Memory OLTP** fails on MI | Only **Business Critical** supports it, not General Purpose |
| Queries slower after migration | Update statistics, review **compatibility level**, compare plans with **Query Store** |

---

## ✅ **Summary**

- **Retired tools:** DMA (Jul 2025), Azure Data Studio + migration extension (Feb 2026), DMS classic SQL scenarios (Mar 2026). Use **DMS**, **Azure Arc migration**, **LRS**, **MI link**, **SSMS**.
- Start troubleshooting from **DMS Monitor migrations**, the **Activity log**, and **LRS / link state** commands.
- Most backup-based failures = **backup format** (append, no CHECKSUM, multiple log files) or **storage permissions** (SAS Read+List only, **Storage Blob Data Reader** for user/MI identity).
- MI link failures = **network** (5022, private path, UCS protocol), **certificates**, or **log truncation during seeding (1408/1412)**.
- Azure SQL Database via DMS is **offline only**, and has specific data-type/table limitations.

---

## 🎯 **Exam Tips / Traps**

- *"Migration to MI fails with error 2009"* → grant the **managed instance's identity** **Storage Blob Data Reader**.
- *"LRS fails to start; SAS has Read, List, Write"* → SAS must be **Read + List only**.
- *"Need to keep adding log backups during LRS"* → **continuous mode**, not autocomplete.
- *"Error 1412 creating MI link"* → log backup truncated the log during seeding → **drop & recreate** the link, pause log backups or use **TF 12381**.
- *"Can't connect to named instance from DMS"* → open **UDP 1434** + the dynamic TCP port.
- *"Minimal downtime migration to Azure SQL Database"* → DMS can't do it (offline only); think **transactional replication** or a third-party CDC tool.
- *"DB must be available immediately after cutover to Business Critical"* → **MI link**, or migrate to **GP** then upgrade.
- *"Migration uses FILESTREAM"* → target **SQL Server on Azure VM**, not MI.

> Source: https://learn.microsoft.com/en-us/previous-versions/azure-data-studio/extensions/azure-sql-migration-extension, https://learn.microsoft.com/en-us/azure/dms/known-issues-troubleshooting-dms, https://learn.microsoft.com/en-us/azure/dms/known-issues-azure-sql-db-managed-instance-online, https://learn.microsoft.com/en-us/sql/sql-server/azure-arc/migrate-to-azure-sql-managed-instance-troubleshoot, https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/log-replay-service-migrate, https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/managed-instance-link-troubleshoot-how-to, https://learn.microsoft.com/en-us/data-migration/sql-server/database/database-migration-service, https://learn.microsoft.com/en-us/sql/dma/dma-overview
