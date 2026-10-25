## 🔍 **What is Log Replay Service (LRS)?**

- **LRS is a log shipping-based migration tool**.
- It enables **controlled, step-by-step migration** of databases from **on-prem SQL Server** to **Azure SQL Managed Instance**.
- Compared to other tools like **Azure Database Migration Service (DMS)**, LRS gives you **more flexibility**, but requires **manual setup**.

---

## 🛡️ **Why Use LRS?**

| When to Use LRS                                                                   | Why                                                        |
| --------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| Need more manual control over the migration                                       | You can control when to apply backups and when to cut over |
| Very low tolerance for downtime                                                   | Continuous mode keeps syncing changes until cutover        |
| Can’t install DMS in your environment                                             | No DMS installation needed                                 |
| Can’t open outbound ports to Azure from on-prem                                   | Uses Azure Blob Storage instead                            |
| On-prem SQL Server can only produce backups (can’t run migration agents directly) | You upload backups to Azure Blob                           |

---

## 🔧 **How LRS Works**

1. **You take backups of your on-premises database** (full, differential, and log backups).
2. **You upload these backups to Azure Blob Storage.**
3. **LRS reads those backups and restores them** into Azure SQL Managed Instance.
4. You can **continue taking backups and uploading them** until ready to cut over.
5. When ready, you finalize the migration (**cutover**) and switch your applications to use the managed instance.

---

## 🔁 **LRS Modes of Operation**

| Mode             | Description                                                                      | Suitable for                        | Backup Chain                                 |
| ---------------- | -------------------------------------------------------------------------------- | ----------------------------------- | -------------------------------------------- |
| **Autocomplete** | You upload all backup files first; LRS restores them and completes migration.    | Passive workloads, planned downtime | Entire backup chain uploaded in advance      |
| **Continuous**   | LRS continuously monitors for new backup files and restores them as they appear. | Active workloads, minimal downtime  | Backup chain can be updated during migration |

➡️ Continuous mode is typically used when you want **ongoing sync during business hours**, then finalize during a brief cutover window.

⚠️ **Important:** All LRS migrations must complete within **30 days**; otherwise, the migration job is canceled.

---

## 🔒 **Securing the Migration**

LRS requires:

- Azure RBAC role: **Subscription Owner, SQL Managed Instance Contributor**, or a custom role with **Microsoft.Sql/managedInstances/databases/** permissions.
- Azure Blob Storage account for storing your backup files.

Blob Storage Configuration:

- If your storage account uses a firewall, you must configure:

  - **MI subnet delegation**
  - **Storage Service Endpoints** for that subnet.

Authentication:

- Use **either a SAS token** or **Managed Identity** to allow the Managed Instance to access the Blob Storage (**not both**).

---

## 🚀 **Performance Tips for Backup & Restore**

✅ **Split large backups into multiple files**
→ Faster parallel reading/writing during restore.

✅ **Enable compression during backup**
→ Smaller backup size, faster upload to Azure Blob Storage.

✅ **Enable CHECKSUM during backup**
→ Avoids extra integrity checks on restore, making restores faster.

---

## 🔑 **Key Takeaways**

✔️ Use **LRS when you need control, flexibility, or your environment limits the use of DMS.**
✔️ Supports **low-downtime migrations** through continuous log replay.
✔️ Requires **manual setup of backups, Blob Storage, and Managed Instance permissions**.
✔️ Recommended for **complex or sensitive migrations** where downtime must be minimized.
✔️ Source: **SQL Server 2012+, all editions** (any host: on-prem, VM, other clouds). The DB stays in **restoring** state (no reads) until cutover. LRS is the only way to restore **differential** backups on MI. 🔄
✔️ Commands: `Start-` / `Get-` / `Complete-` / `Stop-AzSqlInstanceDatabaseLogReplay` or `az sql midb log-replay start|show|complete|stop`. **Azure Arc** migration can also run LRS from the portal. 🔄
✔️ Cutover to **Business Critical** is slow (3 secondary replicas must be seeded) → prefer **MI link** for BC targets. 🔄

---
