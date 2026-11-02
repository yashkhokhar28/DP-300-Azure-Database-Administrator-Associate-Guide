# Migrate to SQL Server on Azure VMs

> **Exam skills:** Implement an offline migration strategy, Implement an online migration strategy, Implement a migration to Azure
> **Reading time:** ~6 min

## The big idea
A SQL Server VM is normal SQL Server. So every on-premises trick works: backup and restore, log shipping, availability groups. Pick by downtime: **hours OK** → backup/restore; **minutes** → DMS or log shipping; **almost none** → distributed AG.

Analogy: moving to an identical house in another town. Everything fits. You only choose how fast you move.

## Downtime and method
| Downtime | Method |
|----------|--------|
| Almost none | **Distributed AG** / Always On AG |
| Minutes | **DMS online**, log shipping, transactional replication, Azure Migrate |
| Hours or days | Backup/restore, detach/attach, BACPAC, bcp |

## Native backup and restore
The simplest method.
1. Back up the database, ideally straight to **Azure Blob Storage** with `BACKUP TO URL`.
2. Restore it on the VM from the same URL.

**Block blob vs page blob:**
| Blob | Credential | Use |
|------|-----------|-----|
| **Block blob** | **SAS token** | Recommended (SQL 2016+). Bigger, faster, safer. |
| Page blob | Storage account key | Older method |

```sql
CREATE CREDENTIAL [https://myacc.blob.core.windows.net/sqlbackups]
  WITH IDENTITY = 'SHARED ACCESS SIGNATURE', SECRET = '<sas token>';
BACKUP DATABASE contoso TO URL = 'https://myacc.blob.core.windows.net/sqlbackups/contoso.bak'
  WITH COMPRESSION, CHECKSUM;
RESTORE DATABASE contoso FROM URL = 'https://myacc.blob.core.windows.net/sqlbackups/contoso.bak'
  WITH NORECOVERY;   -- keep restoring diffs and logs, then WITH RECOVERY
```
- `NORECOVERY` leaves the database **restoring**, so you can add more log backups. This shortens downtime.
- Backups **over 200 GB**: **stripe** over several files/URLs, and compress.
- If you back up to a local file, upload with **AzCopy** (command line) or **Azure Storage Explorer** (GUI).
- SQL on a VM does not "know" it is in Azure. You set up the credential yourself.

**Example:** A 300 GB accounts database moves on a long weekend. Friday: full backup striped over 4 URLs. Saturday: restore with NORECOVERY, then the final log backup, then RECOVERY.

## Distributed availability group
A **distributed AG** joins two availability groups, for example on-premises and Azure. Data flows live until you fail over.
- Source keeps working. Downtime is only the **failover**.
- From a **standalone instance**: no Windows cluster or listener needed.
- From an **existing AG**: needs a Windows failover cluster and listener.
- No shared storage needed.
- Needs **VPN or ExpressRoute** between sites.
- Configure with **T-SQL**. The old "Add Azure Replica Wizard" only worked with classic VMs, which retired in Sep 2023.

**Example:** A global retailer's CRM cannot be down. A distributed AG copies it to Azure VMs for two weeks. Cutover is a planned failover of a few seconds.

## Other methods
| Method | Downtime | When |
|--------|----------|------|
| **DMS** (portal, CLI, PowerShell) | Online or offline | Restores your backups from a share or Blob. Needs a SHIR for network shares |
| **Azure Migrate** | Low | Moves the **whole server** (OS + SQL) as-is |
| **Log shipping** | Low | Ship log backups, final log at cutover. Simpler than AGs |
| **Transactional replication** | Low | No Always On available |
| **Detach/attach from URL** | High | Move `.mdf`/`.ldf` files to Blob, attach on VM |
| **Convert to VHD** | Medium | Upload the whole VM disk. Old versions, system + user databases |
| **BACPAC** | Medium | Small, simple databases |
| **bcp** | High | Table-level data only |
| **Azure Data Factory** | Low/medium | Many sources, transforms |
| **Azure Import/Export service** | Offline | Ship **physical disks** when the network is too slow |

**DMS to a VM, things to know:**
- DMS does **not take backups**. You provide them (compressed is faster).
- Backups on a **network share** need a **SHIR**. You can't reuse an Azure Data Factory SHIR.
- Up to **10 concurrent migrations** per SHIR.
- In Blob Storage, keep each database in **its own folder**.
- You **can't overwrite** an existing database on the target. Drop it or use a new name.

Note: DMA (retired July 2025) could also move **logins and server roles** to a VM. Today, script logins yourself.

## Remember for the exam
- "Near-zero downtime to a SQL VM" → **distributed AG**.
- "Simple, downtime OK" → **backup to URL + restore**.
- Backup to URL: **block blob + SAS** is recommended.
- Over 200 GB → **striped** backups.
- `WITH NORECOVERY` keeps the database ready for more restores.
- "Move the whole server as-is" → **Azure Migrate**.
- "Network too slow for 50 TB" → **Azure Import/Export** (ship disks).
- DMS network share → **SHIR** needed.

## Quick check
1. Which credential type gives a block blob backup?
   <details><summary>Answer</summary>A SAS token (shared access signature). A storage account key gives a page blob.</details>
2. You migrate a standalone SQL Server with a distributed AG. Do you need a Windows cluster on the source?
   <details><summary>Answer</summary>No. Only an existing AG source needs a cluster and listener.</details>
3. Restored database must accept more log backups. Which option do you use?
   <details><summary>Answer</summary>WITH NORECOVERY. Use WITH RECOVERY only on the last restore.</details>
