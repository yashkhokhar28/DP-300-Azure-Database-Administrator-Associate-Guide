## 📦 **What Is MI Database Copy / Move?**

- A built-in, **online** way to copy or move a **user database** from one **Azure SQL Managed Instance to another**.
- Under the hood it uses **Always On availability group technology**. Replication is **asynchronous, near real-time and consistent**.
- DP-300 skill: **"Implement Azure SQL Managed Instance database copy and move"** (under *Plan and implement a migration strategy*).

| Operation | Source DB after completion                         | Typical use                                                  |
| --------- | -------------------------------------------------- | ------------------------------------------------------------ |
| **Copy**  | **Stays online**; replication simply stops          | Refresh dev/test from prod, create a parity copy             |
| **Move**  | **Dropped** after the destination comes online      | Rebalance load, move to an instance with more resources, consolidate |

- **Move is designed for zero data loss.** On completion, the source stops accepting writes, the last transactions replicate, the destination comes online, and only then is the source dropped.
- **Copy**: you trigger completion, but you **don't control the exact moment** log replication stops. After that, both databases are independent and read/write.

---

## 🔁 **Workflow (4 phases)**

1. **Start**: pick the database(s), source MI and destination MI → **seeding** begins.
2. **Ready for completion**: seeding is done. Changes on the source **keep replicating** to the destination.
3. **Complete** (manual): the destination comes online for read/write. Do this **within 24 hours**, or the operation is **auto-cancelled** and the destination DB is dropped.
4. **Move** → the source DB is dropped. **Copy** → the source stays online.

- You can **cancel** at any time before completion. Cancelling stops seeding and drops the destination.
- Seeding speed is **up to ~360 GB/hour** in optimal conditions (global VNet peering). It depends on DB size, workload and the number of parallel operations.

```sql
-- Monitor seeding progress (run on source or destination MI)
SELECT role_desc, transfer_rate_bytes_per_second,
       transferred_size_bytes, database_size_bytes,
       start_time_utc, estimate_time_complete_utc
FROM sys.dm_hadr_physical_seeding_stats;
```

---

## 🛠️ **How to Run It**

**Azure portal:** MI → **Data management → Databases** → select DB(s) → **Copy** or **Move** → source/destination details → **Start** → watch **Operation details** → select **Ready for completion** → **Complete**.

**PowerShell (Az.Sql; v5.1+ for cross-subscription)**

```powershell
# Start a copy (use Move-AzSqlInstanceDatabase for a move)
Copy-AzSqlInstanceDatabase -DatabaseName 'SalesDb' -InstanceName 'mi-prod' -ResourceGroupName 'rg-prod' `
  -TargetInstanceName 'mi-test' -TargetResourceGroupName 'rg-test' -TargetSubscriptionId '<sub-id>'   # TargetSubscriptionId only if cross-sub

Get-AzSqlInstanceDatabaseCopyOperation -DatabaseName 'SalesDb' -InstanceName 'mi-prod' -ResourceGroupName 'rg-prod' `
  -TargetInstanceName 'mi-test' -TargetResourceGroupName 'rg-test'

Complete-AzSqlInstanceDatabaseCopy -DatabaseName 'SalesDb' -InstanceName 'mi-prod' -ResourceGroupName 'rg-prod' `
  -TargetInstanceName 'mi-test' -TargetResourceGroupName 'rg-test'
# Move equivalents: Move-AzSqlInstanceDatabase / Get-AzSqlInstanceDatabaseMoveOperation / Stop-AzSqlInstanceDatabaseMove (cancel)
```

**Azure CLI (2.63.0+ for cross-subscription)**

```bash
az sql midb move start    --name SalesDb -g rg-prod --managed-instance mi-prod --dest-rg rg-new --dest-mi mi-new
az sql midb move list     --name SalesDb -g rg-prod --managed-instance mi-prod
az sql midb move complete --name SalesDb -g rg-prod --managed-instance mi-prod --dest-rg rg-new --dest-mi mi-new
# az sql midb copy start|list|complete|cancel  -> same pattern; --dest-sub-id for cross-subscription
```

**REST API:** `ManagedDatabases – Start Move` (`POST .../databases/{db}/startMove`) with body `destinationManagedDatabaseId` + `operationMode` = **`Move`** (default) or **`Copy`**. Then **Complete Move** or **Cancel Move**.

---

## 📋 **Prerequisites & Limitations**

- **Permissions**: *read* on the source instance's resource group, plus *write* at **database level** on both source and destination instances.
- **Networking**: if the instances are in different VNets, you need **connectivity (e.g., VNet peering)**, with **TCP 5022** and **11000–11999** open inbound/outbound on **both subnets**. These are the same rules as failover groups.
- **Same Azure region only**: cross-region copy/move is **not supported**.
- **Cross-subscription: supported within the same Microsoft Entra tenant.**
- **User databases only.** System databases can't be copied or moved.
- The source and destination instances **can't be the same**. You **can't rename** the DB during the operation.
- One DB = **one copy/move at a time**. A source instance runs **up to 8 operations concurrently**; extra ones are queued.
- **Not supported** if the DB is in a **failover group** or uses **Managed Instance link**. Source/destination instances shouldn't have a failover group configured.
- **PITR backups don't travel.** After a move, the destination starts a **new backup chain**. Database **tags** aren't copied either.
- **TDE with customer-managed key**: the destination instance needs access to the **same Key Vault key**.
- **Update policy**: you can copy/move only to an instance with the **same or a higher** update policy (SQL 2022 → SQL 2025 → Always-up-to-date), never down.
- After a move, **reconfigure** transactional replication, CDC and distributed transactions.

---

## ⚖️ **Copy/Move vs Other Ways to Relocate an MI Database**

| Method                                 | Online?            | Cross-region?        | Cross-subscription?        | Notes                                                                      |
| -------------------------------------- | ------------------ | -------------------- | -------------------------- | -------------------------------------------------------------------------- |
| **Database copy / move**               | ✅ Yes (near-zero downtime) | ❌ Same region | ✅ Same tenant              | **Current state** of the DB; you choose when to cut over                    |
| **Point-in-time restore (PITR)**       | ❌ Restore = new DB | ❌ Same region only  | ✅ Same region + tenant (EA/CSP/PAYG) | DB as of a **past moment** (7–35 days); also restores **deleted** DBs |
| **Geo-restore**                        | ❌                  | ✅ Any region         | –                          | From **geo-replicated backups** (needs GRS backup storage); some data loss possible (backup geo-replication lag) |
| **Failover group** (+ planned failover)| ✅                  | ✅ Yes                | –                          | **Instance-level**: replicates **all user DBs**; a DR feature, not a per-DB move |
| **COPY_ONLY backup to URL + restore**  | ❌ Offline cutover   | ✅                    | ✅                          | Manual; works to SQL Server only if the update policy allows              |
| **Export/Import BACPAC**               | ❌                  | ✅                    | ✅                          | Slow for large DBs; logical copy                                          |

```sql
-- Manual cross-region/offline alternative: copy-only backup to Blob, then RESTORE on target MI
BACKUP DATABASE SalesDb
TO URL = 'https://mystorage.blob.core.windows.net/backups/SalesDb.bak'
WITH COPY_ONLY, COMPRESSION;
-- Note: MI blocks COPY_ONLY backups of DBs encrypted with a *service-managed* TDE key
-- (switch to a customer-managed key or temporarily disable TDE first).
```

---

## ✅ **Summary**

- **Copy** keeps the source online; **move** drops it. Both are **online**, AG-based and **zero-data-loss** (move).
- Flow: **Start → seeding → ready for completion → Complete within 24 h** (or auto-cancel).
- Tools: **portal, PowerShell (`Copy-/Move-AzSqlInstanceDatabase`), CLI (`az sql midb copy|move`), REST (`startMove` + `operationMode`)**.
- Limits: **same region**, cross-subscription only **within the same tenant**, max **8 concurrent** per source, **no failover-group/MI-link DBs**, PITR history doesn't move, update policy can't go down.

---

## 🎯 **Exam Tips / Traps**

- "Move a DB to another MI in the **same region** with **minimal downtime** and **no data loss**" → **database move**.
- "Refresh a **test MI** from **production** while prod stays online" → **database copy**.
- "Copy a DB to an MI in **another region**" → **not** copy/move. Use **geo-restore**, a **failover group**, or backup to URL + restore.
- "Restore the DB **as it was yesterday** on another MI" → **PITR** (copy/move only gives the current state).
- Operation shows **"ready for completion"** but nothing happens → you must **Complete** it. After **24 h** it auto-cancels.
- "Copy/move fails" → check **VNet peering + ports 5022 / 11000–11999**, **failover group membership**, **CMK key access**, and **update policy direction**.
- After a **move**, old **PITR backups are gone**, so take a fresh backup or LTR snapshot before moving if you need history.
- MI **failover groups** cover the **whole instance**. They aren't the tool for moving a single database.

> Source: https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/database-copy-move-how-to · https://learn.microsoft.com/en-us/rest/api/sql/managed-databases/start-move · https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/point-in-time-restore · https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/update-policy
