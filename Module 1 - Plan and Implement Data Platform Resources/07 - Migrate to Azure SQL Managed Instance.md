# Migrate to Azure SQL Managed Instance

> **Exam skills:** Implement an online migration strategy, Implement a migration to Azure, Implement a migration between Azure SQL services, Implement Azure SQL Managed Instance database copy and move
> **Reading time:** ~10 min

## The big idea
Managed Instance understands **native SQL Server backups**. So most MI migrations are "restore backups" (LRS) or "live copy" (MI link). Azure Arc wraps both in one portal flow. Inside Azure, you can **copy or move** a database from one MI to another.

Analogy: moving to a serviced flat that accepts your own furniture. You can send it in boxes (backups), or keep a live video link until moving day (MI link).

## Your options
| Method | Online? | Notes |
|--------|---------|-------|
| **Log Replay Service (LRS)** | Yes (continuous mode) | Restores your backups from Blob Storage |
| **Managed Instance link** | Yes | Near real-time replication using a distributed AG |
| **SQL Server migration in Azure Arc** | Yes | Portal flow that uses MI link or LRS |
| **Native backup/restore** | No | `RESTORE FROM URL` of a full backup |
| **Transactional replication** | Both | MI can be publisher, distributor or subscriber |
| **BACPAC** | No | SqlPackage or SSMS only (no portal import to MI) |

**Example:** A sports retailer's catalog database feeds the website, sales app and reports. It uses SQL Agent and cross-database joins. They pick MI so no app code changes.

## Connect on-premises to MI
- MI lives in a **private subnet**. On-premises reaches it by **site-to-site VPN** or **ExpressRoute**. Point-to-site VPN is for one admin machine.
- Optional **public endpoint** on port **3342**. A DBA turns it on; the network team opens the port.

## Native backup and restore
```sql
CREATE CREDENTIAL [https://acct.blob.core.windows.net/backups]
  WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = '<sas token>';
RESTORE DATABASE Sales FROM URL = 'https://acct.blob.core.windows.net/backups/Sales.bak';
```
MI restore limits:
- Only from **Azure Blob Storage** (no local disk).
- Restores single databases, not a whole instance. Can't overwrite an existing database.
- No **FILESTREAM/FileTable** backups.
- Active **In-Memory OLTP** objects → **Business Critical** only, not General Purpose.
- **TDE** backups need the certificate or key on MI first.
- To apply **differential** backups you need LRS.

## Log Replay Service (LRS)
LRS is log shipping into MI. You take backups; LRS restores them.
1. Take full, differential and log backups. Upload them to **Blob Storage**.
2. Start LRS. It restores what it finds.
3. Keep uploading log backups.
4. **Cutover**: complete the migration and switch apps.

| Mode | How | Use |
|------|-----|-----|
| **Autocomplete** | Upload the **whole chain first**, give the last file name. LRS finishes by itself. | Planned downtime |
| **Continuous** | LRS keeps watching for new files. You complete it manually. | Minimal downtime |

Key facts:
- Source: **SQL Server 2012+, all editions**, any host.
- The database stays in **restoring** state (not readable) until cutover.
- Must finish within **30 days**, or the job is cancelled.
- Access to storage: **SAS token or managed identity**, not both. SAS = **Read + List only**.
- Commands: `Start-/Get-/Complete-/Stop-AzSqlInstanceDatabaseLogReplay` or `az sql midb log-replay start|show|complete|stop`.
- Backups: **one backup per file**, `WITH CHECKSUM`, `COMPRESSION`, split big backups into several files.
- Cutover to **Business Critical** is slow (it must seed 3 replicas). Prefer MI link for BC.

When LRS fits: you need full control, can't install DMS, or the source can't open ports to Azure. LRS only needs Blob Storage.

**Example:** A 2014 Standard Edition server can't use MI link. LRS in continuous mode keeps MI up to date for a week. Cutover on Saturday takes minutes.

## Managed Instance link
MI link uses a **distributed availability group** to replicate one database from SQL Server to MI in near real time.

| SQL Server version | Direction |
|-------------------|-----------|
| 2016 SP3, 2017 CU31, 2019 CU20 (2016/2017 also need the Azure Connect pack) | **One-way.** Failover to MI breaks the link; no failback. |
| **2022, 2025** | **Two-way.** Online failover **and failback**. MI must use the matching update policy (not Always-up-to-date). |

Key facts:
- **One database = one link** (a multi-database mode is in preview).
- Editions: **Enterprise, Standard, Developer**.
- Needs **private** connectivity (VPN, ExpressRoute or VNet peering). It uses the MI VNet-local endpoint, port **5022**.
- MI DB is **readable** during migration (good for reports).
- **No time limit.** Cutover takes seconds.
- An MI with a link **can't be in a failover group**.
- No FILESTREAM or FileTable.
- Set up with the **SSMS wizard**, T-SQL or PowerShell.
- Also used for **DR** (MI as standby) and **read offload**.

## LRS vs MI link
| | MI link | LRS |
|-|---------|-----|
| Technology | Distributed AG | Backup restore from Blob |
| Source | SQL Server 2016+ | SQL Server 2012+ |
| Editions | Enterprise, Standard, Developer | All |
| Target readable during migration | Yes | No |
| Time limit | None | 30 days |
| Best for | Business Critical, seconds of downtime | Old versions, Web/Express, public endpoint |
| Reverse migration | Possible (2022/2025) | No |

## SQL Server migration in Azure Arc
For an **Arc-enabled** SQL Server, the portal has **Migration → Database migration** with four tiles:
1. **Assess source instance:** blockers and a right-sized MI. Runs **every weekend** automatically, or on demand.
2. **Select target:** pick or create an MI.
3. **Migrate data:** choose **MI link** (real-time) or **LRS** (log shipping).
4. **Monitor and cutover:** watch status and **replication lag**, then cut over.

- It uses **only MI link or LRS** (not DMS, not BACPAC).
- MI link here: up to **10** databases at once (recent Azure extension for SQL Server). LRS: unlimited.
- For LRS you must first upload at least one **full backup**.
- **Copilot** can explain the assessment and run steps.
- **Failover cluster instances can't be migrated** this way.
- No reverse migration in the Arc flow.

**Example:** The bike company Arc-enabled its servers last year. The DBA opens the portal, reads the weekend assessment, creates the recommended MI and starts an MI link. Cutover is a click.

## MI database copy and move
Copy or move a **user database** from one MI to another, **online**, using Always On technology.

| Operation | Source after completion | Typical use |
|-----------|------------------------|-------------|
| **Copy** | Stays online | Refresh test from production |
| **Move** | **Dropped** | Rebalance load, consolidate instances |

Steps: **Start** → seeding → **Ready for completion** (changes keep flowing) → **Complete** (you click). You must complete within **24 hours**, or it auto-cancels and drops the target.

```powershell
Copy-AzSqlInstanceDatabase -DatabaseName SalesDb -InstanceName mi-prod -ResourceGroupName rg-prod `
  -TargetInstanceName mi-test -TargetResourceGroupName rg-test
Complete-AzSqlInstanceDatabaseCopy -DatabaseName SalesDb -InstanceName mi-prod -ResourceGroupName rg-prod `
  -TargetInstanceName mi-test -TargetResourceGroupName rg-test
# Move: Move-AzSqlInstanceDatabase. CLI: az sql midb copy|move start|complete|cancel
```

Rules:
- **Same region only.** Cross-subscription OK within the **same Entra tenant**.
- User databases only. Source and target instance must differ. No rename.
- Up to **8** operations at once per source instance.
- **Not** for databases in a **failover group** or with **MI link**.
- Different VNets need peering and ports **5022** and **11000–11999** open.
- **PITR backups don't move.** The target starts a new backup chain.
- TDE with customer-managed key → target needs the **same Key Vault key**.
- Target **update policy** must be the same or higher.
- Move is designed for **zero data loss**.

**Other ways to relocate an MI database:**
| Need | Use |
|------|-----|
| Another region | Geo-restore, failover group, or COPY_ONLY backup to URL + restore |
| As it was yesterday | Point-in-time restore |
| Whole instance for DR | Failover group |

## Remember for the exam
- "SQL Server 2012, Web edition, to MI" → **LRS**.
- "Keep adding log backups" → LRS **continuous**, not autocomplete. LRS limit = **30 days**.
- "Readable target, Business Critical, seconds of downtime" → **MI link**.
- "Fail back to on-premises" → MI link with SQL Server **2022/2025**.
- "Arc-enabled server, migrate from the portal" → **Arc migration** (uses MI link or LRS).
- "Refresh test MI from prod, prod stays online" → **database copy**.
- "Move to MI in another region" → **not** copy/move. Use geo-restore or backup/restore.
- "Ready for completion, nothing happens" → you must click **Complete** within **24 h**.

## Quick check
1. Your source is SQL Server 2019 Enterprise. You need a readable MI copy during migration. LRS or MI link?
   <details><summary>Answer</summary>MI link. LRS keeps the database in restoring state, so it can't be read.</details>
2. A copy operation is "Ready for completion" for two days, then disappears. Why?
   <details><summary>Answer</summary>It wasn't completed within 24 hours, so it was auto-cancelled.</details>
3. Which two engines does Azure Arc migration use?
   <details><summary>Answer</summary>Managed Instance link and Log Replay Service.</details>
