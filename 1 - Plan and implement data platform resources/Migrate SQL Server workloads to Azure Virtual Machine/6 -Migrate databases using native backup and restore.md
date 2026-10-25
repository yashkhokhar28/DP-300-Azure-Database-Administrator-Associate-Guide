## ✅ **What is Backup and Restore Migration?**

It’s one of the **simplest ways to migrate SQL Server databases**. The process is almost identical to what you do on-premises:

1. **Take a database backup**.
2. **Copy the backup file to the destination Azure VM**.
3. **Restore the database** on the SQL Server running inside the Azure VM.

---

## 📦 **Azure Blob Storage for Backups**

When migrating to Azure, you often use **Azure Blob Storage** to temporarily hold the backup files instead of manually copying them to the Azure VM. SQL Server can back up directly to and restore from Azure Blob Storage.

### 🔹 **Blob Types:**

| Blob Type      | Authentication                | Preferred for                                                                               |
| -------------- | ----------------------------- | ------------------------------------------------------------------------------------------- |
| **Block blob** | Shared Access Signature (SAS) | ✔ Recommended for SQL Server 2016+<br>✔ Larger backups<br>✔ Better security and performance |
| **Page blob**  | Storage account key           | Older methods                                                                               |

---

## 🛡️ **Why Use Block Blob?**

✔ Safer authorization using **SAS token**
✔ Supports **larger backup files**
✔ Better **backup performance**

---

## 🔑 **How SQL Server Connects to Blob Storage**

You need to **create a SQL Server credential** to allow SQL Server to connect to Azure Blob Storage.

Two ways to authenticate:

1. **Storage Account Key** → stores the backup as a **page blob**
2. **Shared Access Signature (SAS)** → stores the backup as a **block blob** (**Recommended**)

---

## 🔨 **Backup Example (To Azure Blob Storage)**

```sql
BACKUP LOG contoso
TO URL = 'https://myacc.blob.core.windows.net/mycontainer/contoso202003271200.trn'
```

---

## 🔧 **Restore Example (From Azure Blob Storage)**

```sql
RESTORE DATABASE contoso
FROM URL = 'https://myacc.blob.core.windows.net/mycontainer/contoso20200327.bak'
WITH NORECOVERY
```

- `NORECOVERY` means the database is left **in a restoring state** so you can apply additional differential or log backups later.

---

## ⚙️ **Backup and Restore: Azure VM vs. On-Premises**

Even though your SQL Server is running on an Azure VM, it doesn’t **automatically know it’s in Azure**. You must set up Blob Storage access explicitly.

---

## 🚀 **How to Transfer Backup Files to Azure VM**

- **Recommended:** Back up directly to Azure Blob Storage using `BACKUP TO URL`.
- Alternative: Back up to a local file, then copy using:

  - **AzCopy CLI tool**: Fast, command-line upload/download from/to Azure Blob Storage.
  - **Azure Storage Explorer**: GUI method. (The Learn unit says "Azure Data Explorer", but that's a Kusto analytics service, not a file-copy tool.) 🔄

---

## ⚙️ **Performance Optimization Tips**

| Scenario                  | Recommendation                              |
| ------------------------- | ------------------------------------------- |
| Large database backups    | ✔ Compress the backup (`WITH COMPRESSION`)  |
| Backup larger than 200 GB | ✔ Use striped backups (multiple files/URLs) |

---

## 🔍 **Backup URL Format**

```plaintext
https://{AccountName}.blob.core.windows.net/{ContainerName}/{BackupFile}
```

Example:

```plaintext
https://myaccount.blob.core.windows.net/sqlbackups/contoso20200327.bak
```

---

## ➕ **Bonus Tip**

You can **organize backups in folders** inside a container:

- `/FULL` → Full backups
- `/DIFF` → Differential backups
- `/LOG` → Transaction logs

---

## 🟢 **Summary**

✔ Simple & well-known method
✔ Suitable for small to medium databases or one-time migrations
✔ Uses **Azure Blob Storage for cloud-based backups**
✔ Supports restoring **striped backups** and large databases efficiently
✔ Easy to automate with SQL Server Agent jobs and scripts

---
