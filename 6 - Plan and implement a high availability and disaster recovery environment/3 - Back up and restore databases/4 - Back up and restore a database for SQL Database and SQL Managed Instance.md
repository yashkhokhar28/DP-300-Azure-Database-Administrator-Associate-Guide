## 🔍 **Main Idea**

In Azure's **Platform as a Service (PaaS)** databases (SQL Database and SQL Managed Instance), **backups happen automatically**. Microsoft manages them for you, providing built-in high availability and disaster recovery.

---

## 🔑 **Automatic Backup Process**

For **both Azure SQL Database and SQL Managed Instance**:

- ✔️ **Full backup:** every **week**
- ✔️ **Differential backup:** every **12 or 24 hours** (SQL DB default: 12 h for vCore, 24 h for DTU) 🔄
- ✔️ **Transaction log backup:** approximately every **10 minutes** 🔄
- ✔️ **Hyperscale** uses **storage snapshots** instead of full/differential/log backups (near-instant backups, fast restores). 🔄

By **default**, backups are stored in **geo-redundant storage (GRS)**, copied to the paired region to protect against regional outages. You can choose **LRS**, **ZRS**, **GRS**, or **GZRS**. **Geo-restore** works only with GRS or GZRS. For Hyperscale, you can set backup redundancy only when you create the database. 🔄

---

## 🔐 **Key Features**

### ✅ **Point-in-Time Restore (PITR)**

- You can restore to **any point within the backup retention period.**
- Default retention: **7 days**, configurable from **1 to 35 days** (**Basic** tier: 1–7 days). 🔄
- You restore the database as a **new database** (restore in place is **not supported**).

**How to restore:**

- Azure Portal
- Azure CLI (`az sql db restore`)
- PowerShell
- REST API

---

### ✅ **Restore Deleted Database**

- If you accidentally **delete a database**, you can restore it from the backups taken **before deletion**.
- You’ll find deleted databases in the **"Deleted Databases"** section of the server on the Azure portal.
- You must **give the restored database a new name.**

⚠️ If the entire **SQL Server resource** (logical server) is deleted, its databases and PITR backups are **lost permanently**, but **LTR backups are kept** and can be restored to another server. For a deleted managed instance, contact support within **5 days**. 🔄

---

### ✅ **Accelerated Database Recovery (ADR)**

- ADR speeds up **recovery of databases** from crashes or long-running transactions.
- Enabled by default in Azure SQL.
- Helps reduce downtime during recovery.
- Also available in **SQL Server 2019+ on-premises.**

---

## 🔁 **Manual Backup and Restore on Managed Instance**

Azure SQL **Managed Instance only**, not SQL Database, allows **manual backups**:

- Backup to Azure Blob Storage using the `BACKUP DATABASE ... TO URL` syntax.
- **Copy-only backups** are required because Azure manages the backup chain. 🔄
- A database encrypted with a **service-managed TDE key** can't be backed up with `COPY_ONLY`. Use a **customer-managed TDE key** instead. 🔄

Example backup:

```sql
BACKUP DATABASE contoso
TO URL = 'https://myacc.blob.core.windows.net/mycontainer/contoso.bak'
WITH COPY_ONLY
```

⚠️ **You can’t restore backups from SQL Managed Instance to SQL Database**, or vice versa.
Each environment has its own backup system.

---

## 📊 **Comparison: SQL Database vs Managed Instance**

| Feature                       | Azure SQL Database | Azure SQL Managed Instance |
| ----------------------------- | ------------------ | -------------------------- |
| Automatic backups             | ✅ Yes             | ✅ Yes                     |
| Point-in-time restore         | ✅ Yes             | ✅ Yes                     |
| Backup to URL                 | ❌ No              | ✅ Yes                     |
| Manual restore from URL       | ❌ No              | ✅ Yes                     |
| Accelerated Database Recovery | ✅ Yes             | ✅ Yes                     |
| Restore in place              | ❌ No              | ❌ No                      |
| Restore deleted database      | ✅ Yes             | ✅ Yes                     |

---

## 🚨 **Important Notes**

- **SQL Database** backups can't be restored to SQL Server. **SQL MI** copy-only backups **can** be restored to **SQL Server 2022 or 2025** when the MI **update policy** matches. SQL Server native backups **can** be restored to MI with `RESTORE ... FROM URL`. 🔄
- If you want **longer backup retention (beyond 35 days)** for compliance, configure **Long-Term Retention (LTR)** policies (weekly, monthly, and yearly full backups kept for up to **10 years**). 🔄

---
