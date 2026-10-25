## ✅ **What This Is About**

When you run **SQL Server on an Azure VM**, **you are responsible** for setting up and managing backups—unlike Azure SQL Database (PaaS), where backups are automatic.

You need to **back up both types of databases:**

- **System databases** → e.g., `master`, `msdb`, `model` (hold SQL Server’s own config and jobs).
  ➔ _Usually backed up but not restored onto other servers._
- **User databases** → Your actual application data.
  ➔ _Main priority for backup/restore._

---

## 🔁 **Types of SQL Server Backups**

1. **Full Backup**

   - Takes a complete snapshot of the database at that moment.
   - Needed as the base for restores.
   - Example: Nightly backup.

2. **Differential Backup**

   - Captures changes **since the last full backup**.
   - Quicker than a full backup.
   - Example: Taken every few hours.

3. **Transaction Log Backup**

   - Captures all database transactions since the last log backup.
   - Allows **point-in-time recovery**.
   - Also **truncates** the log (marks inactive log space for reuse; it doesn't shrink the file), which stops the log file from growing too large. 🔄
   - Can be scheduled as frequently as needed (e.g., every 15 minutes or less).

---

## 🔔 **Key Recovery Options**

If you want **point-in-time recovery** (e.g., restore to 10:35 AM due to an accidental delete):

- You must combine a **full backup**, the latest **differential backup**, and **transaction log backups**.

Restoring sequence example:

1. Restore **Full Backup** → `WITH NORECOVERY`
2. Restore **Differential Backup (optional)** → `WITH NORECOVERY`
3. Restore **Transaction Log Backups** → `WITH NORECOVERY`
4. When the last backup is restored → **Restore with RECOVERY**.

If you restore with `RECOVERY` too early, you can't apply more backups afterward.

---

## ⚙️ **Recovery Models**

SQL Server databases have **Recovery Models**, which affect what backups are possible:

| Recovery Model  | Backup Types Supported                      | Point-in-Time Restore?       |
| --------------- | ------------------------------------------- | ---------------------------- |
| **FULL**        | Full, Differential, Transaction Log         | ✅ Yes                       |
| **SIMPLE**      | Full, Differential only                     | ❌ No Transaction Log backup |
| **BULK_LOGGED** | Like FULL but optimized for bulk operations | ✅ Yes (with some limits)    |

If your goal is to **minimize data loss (low RPO)**, use the **FULL recovery model**.

---

## 🛡️ **Why This Matters in Azure VM**

- Azure VMs **don’t automatically back up SQL databases** unless you configure it.
- You can:

  - Use SQL Server’s built-in backup features to **back up to Azure Blob Storage**.
  - Use **Azure Backup** or third-party backup tools.
  - Schedule backups and transaction log truncation.

---
