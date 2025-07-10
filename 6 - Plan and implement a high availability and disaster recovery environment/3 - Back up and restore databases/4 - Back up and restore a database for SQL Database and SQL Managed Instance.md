## 🔍 **Main Idea**

In Azure's **Platform as a Service (PaaS)** databases (SQL Database and SQL Managed Instance), **backups happen automatically**. Microsoft manages them for you, providing built-in high availability and disaster recovery.

---

## 🔑 **Automatic Backup Process**

For **both Azure SQL Database and SQL Managed Instance**:

- ✔️ **Full backup:** every **week**
- ✔️ **Differential backup:** every **12 hours**
- ✔️ **Transaction log backup:** every **5–10 minutes**

Backups are stored in **geo-redundant Azure Blob Storage (RA-GRS)**, meaning they are automatically copied to a paired Azure region to protect against regional outages.

---

## 🔐 **Key Features**

### ✅ **Point-in-Time Restore (PITR)**

- You can restore to **any point within the backup retention period.**
- Default retention: **7 days**, extendable up to **35 days**.
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

⚠️ If the entire **SQL Server resource** (logical server) is deleted, **all backups are lost permanently.**

---

### ✅ **Accelerated Database Recovery (ADR)**

- ADR speeds up **recovery of databases** from crashes or long-running transactions.
- Enabled by default in Azure SQL.
- Helps reduce downtime during recovery.
- Also available in **SQL Server 2019+ on-premises.**

---

## 🔄 **Manual Backup and Restore on Managed Instance**

Azure SQL **Managed Instance only**, not SQL Database, allows **manual backups**:

- Backup to Azure Blob Storage using the `BACKUP DATABASE ... TO URL` syntax.
- **Copy-only backups** are required because Azure is managing the transaction log.

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

- You **cannot use Azure SQL PaaS backups to restore to on-premises SQL Server**, and vice versa.
- If you want **longer backup retention (beyond 35 days)** for compliance, configure **Long-Term Retention (LTR)** policies.

---
