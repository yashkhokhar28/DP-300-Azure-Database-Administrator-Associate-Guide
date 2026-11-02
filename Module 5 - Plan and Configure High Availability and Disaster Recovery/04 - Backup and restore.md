# Backup and restore

> **Exam skills:** Recommend a database backup and restore strategy, Perform a database backup by using native tools, Perform a database restore by using native tools, Perform a database restore to a point in time, Configure long-term backup retention, Backup and restore a database by using T-SQL, Backup to and restore from cloud storage
> **Reading time:** ~15 min

## The big idea
Backups are the **last line of defence**. HA and DR copy mistakes too: if someone deletes a table, every replica deletes it. Only a backup lets you go back in time. On a VM, **you** set up backups. In SQL Database and SQL MI, **Azure** takes them automatically.

Think of a diary you photocopy. A full copy every Sunday, the changed pages every day, and a photo of each new line every few minutes. With all three you can rebuild the diary as it was at any moment.

## Backup types
| Type | What it holds | Example schedule |
|---|---|---|
| **Full** | The whole database. The base for every restore | Every night or every week |
| **Differential** | Changes **since the last full** backup | Every few hours |
| **Transaction log** | All changes **since the last log** backup. Allows **point-in-time** restore | Every 5–15 minutes |

- A log backup also **truncates** the log (marks old space as reusable; it does **not** shrink the file). Without log backups, the log file keeps growing.
- **COPY_ONLY** = an extra backup that does **not** change the normal backup chain. Use it for one-off copies.

## Recovery models
| Model | Log backups? | Point-in-time restore? | Use for |
|---|---|---|---|
| **FULL** | Yes | Yes | Production, low RPO |
| **BULK_LOGGED** | Yes | Yes, with limits (not inside a bulk operation) | Large bulk loads |
| **SIMPLE** | **No** | **No** | Dev/test |

**Example:** A payroll database must lose at most 15 minutes of data. It uses the FULL model with log backups every 15 minutes.

## Native T-SQL backup and restore (SQL Server on VM)
Restore order for a point-in-time restore:
1. Restore the **full** backup `WITH NORECOVERY`.
2. Restore the latest **differential** (optional) `WITH NORECOVERY`.
3. Restore each **log** backup in order `WITH NORECOVERY`. On the last one use `STOPAT` for an exact time.
4. Finish `WITH RECOVERY` (brings the database online).

If you use `RECOVERY` too early, you **can't** apply more backups.

```sql
BACKUP DATABASE Sales TO DISK = 'F:\Backup\Sales_full.bak' WITH COMPRESSION, CHECKSUM;
BACKUP LOG Sales TO DISK = 'F:\Backup\Sales_log1.trn';

RESTORE DATABASE Sales FROM DISK = 'F:\Backup\Sales_full.bak' WITH NORECOVERY, REPLACE;
RESTORE LOG Sales FROM DISK = 'F:\Backup\Sales_log1.trn'
  WITH STOPAT = '2026-10-29 10:34:00', RECOVERY;
```

### Tail-log backup
Before you restore over a damaged database, back up the **tail of the log** (the changes made since the last log backup). Without it, those last minutes are lost.
```sql
BACKUP LOG Sales TO DISK = 'F:\Backup\Sales_tail.trn' WITH NORECOVERY;   -- add CONTINUE_AFTER_ERROR if damaged
```
`NORECOVERY` here leaves the database in the restoring state, ready for the restore sequence.

Useful options: **COPY_ONLY**, **COMPRESSION**, **ENCRYPTION** (needs a certificate), **CHECKSUM**, **MOVE** (put files in new paths), **STATS** (show progress).

**Example:** At 10:35 someone runs `DELETE FROM Orders` with no WHERE. The DBA restores the full backup and logs to a new database with `STOPAT 10:34`, then copies the lost rows back.

Native tools: you can also back up and restore in **SSMS** (right-click database → Tasks → Back Up / Restore). Note: Azure Data Studio was retired on 28 Feb 2026, but old questions may still mention it.

## Backup to URL (Azure Blob Storage)
SQL Server can back up **straight to Azure Blob Storage** and restore from it. This is the answer for "backup to and restore from cloud storage". It works for SQL Server on VMs, on-premises SQL Server, and SQL MI.

Path format: `https://<account>.blob.core.windows.net/<container>/<file>.bak`

Two ways to sign in to storage:
| Credential | Blob type | Notes |
|---|---|---|
| **Shared Access Signature (SAS)** | **Block blob** | **Recommended**: cheaper, more secure, can stripe |
| Storage account name + key | Page blob | Older method. One backup max **1 TB** |

```sql
CREATE CREDENTIAL [https://myacc.blob.core.windows.net/sqlbackups]
  WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = '<SAS token without ?>';

BACKUP DATABASE Contoso TO URL = 'https://myacc.blob.core.windows.net/sqlbackups/contoso.bak';
BACKUP LOG Contoso TO URL = 'https://myacc.blob.core.windows.net/sqlbackups/contoso_log.trn';
RESTORE DATABASE Contoso FROM URL = 'https://myacc.blob.core.windows.net/sqlbackups/contoso.bak'
  WITH NORECOVERY;
```

- With SAS, the credential **name is the container URL** and the identity is `SHARED ACCESS SIGNATURE`.
- One block blob holds about **195 GB**. For bigger backups, **stripe** across up to **64 URLs** (about 12.8 TB).
- **SQL Server 2022+** can also back up to **S3-compatible storage** (`TO URL = 's3://...'`, credential identity `S3 Access Key`).
- In SSMS, choose **URL** as the backup or restore media.

**Example:** A logistics firm backs up its on-premises database to Blob Storage with a SAS token each night. During a DR drill it restores the backup onto an Azure VM.

## Backup options for SQL Server on Azure VMs
Azure does **not** back up SQL databases on a VM unless you set it up.

| Option | Restore one database? | Best for |
|---|---|---|
| **Azure VM backup** (snapshot of the whole VM, VSS app-consistent) | No (whole VM/disks) | Full VM recovery, ransomware |
| **Azure Backup for SQL Server in Azure VMs** (workload-aware) | Yes, with point-in-time | Central, vault-based backups and long retention |
| **Automated Backup** (SQL IaaS Agent extension) | Yes | Simple automatic backups to your storage account |
| **Native backups to disk or Azure Files** | Yes | Normal SQL backups you control |
| **Native backup to URL** | Yes | Cloud storage backups you control |

Details:
- **Azure Backup for SQL Server in Azure VMs** installs the `AzureBackupWindowsWorkload` extension and streams native full, differential and log backups to a **Recovery Services vault**. Log backups as often as every **15 minutes**. Snapshot-based backup of SQL instances is in **preview**.
- **Automated Backup** uses the **SQL IaaS Agent extension** (VM registered as a **SQL virtual machines** resource) and configures **Managed Backup to Microsoft Azure**. Retention **1–90 days** (default 90). Log backups every **5–60 minutes**. System databases and encryption are optional. Needs **SQL Server 2016+** (Standard, Enterprise, Developer), **FULL** recovery model, a default or single named instance, and a **Windows** VM. It does not restore for you: restore with T-SQL or SSMS.
- If VM snapshot backups cause problems, set the registry value `USEVSSCOPYBACKUP = TRUE` under `HKLM\SOFTWARE\MICROSOFT\BCDRAGENT` so the VSS backup is copy-only.
- Don't write backups to the **temporary disk** (D:). It is wiped.
- **Don't mix** log backup methods (for example Automated Backup **and** your own log backup job). It breaks the **log chain** and restores fail. Pick **one** tool for log backups.

## Automatic backups in SQL Database and SQL MI
| Backup | How often |
|---|---|
| Full | Every **week** |
| Differential | Every **12 or 24 hours** (SQL DB default: 12 h vCore, 24 h DTU) |
| Log | About every **10 minutes** |

- **Hyperscale** uses **storage snapshots** instead (near-instant backups, fast restores).
- **Short-term retention** (for PITR): default **7 days**, set **1–35 days** (**Basic tier: 1–7 days**).
- SQL Database: **no** user `BACKUP` command. Use automatic backups, or export a **BACPAC** (a file with schema and data) for a copy.
- **SQL MI** allows manual `BACKUP ... TO URL` **only `WITH COPY_ONLY`**, because Azure owns the backup chain. A database using a **service-managed TDE key** can't be copy-only backed up. Use a **customer-managed TDE key** instead.

```sql
-- SQL Managed Instance only
BACKUP DATABASE Contoso TO URL = 'https://myacc.blob.core.windows.net/mi/contoso.bak' WITH COPY_ONLY;
```

Moving backups between products:
- SQL Server native backup → SQL MI: **yes** (`RESTORE ... FROM URL`).
- SQL MI copy-only backup → **SQL Server 2022 or 2025**: yes, when the MI **update policy** matches.
- SQL Database backup → SQL Server: **no**. SQL MI ↔ SQL Database: **no**.

## Backup storage redundancy
| Option | Copies | Survives | Notes |
|---|---|---|---|
| **LRS** (locally redundant) | 3 in **one datacenter** | Disk/rack failure | Cheapest. Not for DR |
| **ZRS** (zone redundant) | 3 across **zones**, one region | Datacenter (zone) failure | Not a regional outage |
| **GRS** (geo redundant) | 3 local + 3 in the **paired region** | Regional outage | Secondary not readable until failover |
| **RA-GRS** (read-access GRS) | Same as GRS | Regional outage | Secondary readable any time |
| **GZRS** | ZRS + paired region | Zone and regional outage | |

- SQL DB/MI backups default to **geo-redundant** storage. You can choose LRS, ZRS, GRS or GZRS.
- **Geo-restore works only with GRS or GZRS.**
- For **Hyperscale**, backup redundancy can be set only when you create the database.

**Example:** A small test database uses LRS to save money. The production database keeps GRS so it can be geo-restored if the region is lost.

## Point-in-time restore (PITR) in PaaS
- Restore to **any second within the retention period**.
- Always restores as a **new database** (no in-place restore). Then rename or repoint the app.
- Tools: portal (database → **Restore**), Azure CLI, PowerShell, REST API.
- SQL MI: restore to the same or another instance. PowerShell gives the fullest support.

```bash
az sql db restore -g rg-prod -s sql-prod -n SalesDb --dest-name SalesDb_0934 \
  --time "2026-10-29T09:34:00Z"
```

```powershell
Restore-AzSqlDatabase -FromPointInTimeBackup -PointInTime "2026-10-29 09:34" `
  -ResourceGroupName rg-prod -ServerName sql-prod -TargetDatabaseName SalesDb_0934 -ResourceId $db.ResourceId
```

**Example:** At 09:35 a faulty release overwrites prices. The DBA restores `SalesDb` to 09:34 as `SalesDb_0934` and copies the correct prices back.

### Restore a deleted database
- Deleted databases show under the server's **Deleted databases** blade. Restore to the deletion time with a **new name**.
- If the **logical server is deleted**, its databases and PITR backups are **lost**. **LTR backups are kept** and can be restored to another server.
- For a deleted **managed instance**, contact support within **5 days**.

### Geo-restore
- Restores from the **geo-replicated backups** to **any server in any region**.
- Used when the whole region is down and you have no geo-replica.
- Longest RTO and higher RPO than geo-replication, because backups are copied with a delay. Needs GRS or GZRS backup storage.

## Long-term retention (LTR)
- Keeps **full backups** for longer than 35 days, up to **10 years**, for compliance.
- Policy has four settings: **weekly (W)**, **monthly (M)**, **yearly (Y)** retention, and **WeekOfYear** (which week's backup is kept as the yearly one).
- Set in the portal (server → **Backups** → **Retention policies**), PowerShell or CLI. Works for SQL DB and SQL MI.
- On an MI failover group, set the LTR policy on **both** instances.

```bash
az sql db ltr-policy set -g rg-prod -s sql-prod -n SalesDb \
  --weekly-retention P8W --monthly-retention P12M --yearly-retention P10Y --week-of-year 1
az sql db ltr-backup list --location westeurope --server sql-prod --database SalesDb
```

```powershell
Set-AzSqlDatabaseBackupLongTermRetentionPolicy -ResourceGroupName rg-prod -ServerName sql-prod `
  -DatabaseName SalesDb -WeeklyRetention P8W -MonthlyRetention P12M -YearlyRetention P10Y -WeekOfYear 1
Restore-AzSqlDatabase -FromLongTermRetentionBackup -ResourceId $ltr.ResourceId ...
```

**Example:** A law firm must keep case data for 7 years. It sets yearly retention `P7Y` and monthly `P12M`.

## Recommending a strategy
| Need | Recommend |
|---|---|
| Undo a mistake from 20 minutes ago (PaaS) | **PITR** |
| Keep backups 7 years (PaaS) | **LTR** |
| Region lost, no geo-replica (PaaS) | **Geo-restore** (GRS/GZRS backups) |
| SQL VM, point-in-time per database, central vault | **Azure Backup for SQL Server in Azure VMs** |
| SQL VM, simple automatic backups to storage | **Automated Backup** (SQL IaaS Agent extension) |
| Whole VM back after ransomware | **Azure VM backup** |
| Cloud storage for on-premises or VM backups | **Backup to URL** with SAS |
| Copy of an MI database to SQL Server 2022 | MI **COPY_ONLY** backup to URL |

Always **test restores**. A backup is only proven when it restores (lesson 5).

## Remember for the exam
- Point-in-time restore needs **FULL** (or BULK_LOGGED) and **log backups**. SIMPLE can't.
- Restore order: full → diff → logs, all `NORECOVERY`, last one `RECOVERY` (`STOPAT` for a time).
- Backup to URL: **SAS + block blobs** recommended. Credential name = container URL.
- SQL MI manual backups must be **COPY_ONLY**. SQL Database has **no** user backups.
- PaaS: full weekly, diff 12/24 h, log ~10 min. PITR **1–35 days**, default **7**. Basic max 7.
- PITR and deleted-database restores always create a **new** database.
- **LTR = up to 10 years**, W/M/Y + WeekOfYear.
- **Geo-restore needs GRS or GZRS** backup storage.
- Don't mix log backup tools on one VM: it breaks the log chain.

## Quick check
1. A database uses the SIMPLE recovery model. Can you restore it to 10:15 today?
   <details><summary>Answer</summary>No. SIMPLE has no log backups, so you can only restore to the time of a full or differential backup.</details>
2. You must take a manual backup of a SQL Managed Instance database to Blob Storage. Which option is required?
   <details><summary>Answer</summary>WITH COPY_ONLY, because Azure owns the normal backup chain on MI.</details>
3. Backup storage for an Azure SQL database is set to LRS. The region goes down. Can you geo-restore?
   <details><summary>Answer</summary>No. Geo-restore works only with GRS or GZRS backup storage.</details>
