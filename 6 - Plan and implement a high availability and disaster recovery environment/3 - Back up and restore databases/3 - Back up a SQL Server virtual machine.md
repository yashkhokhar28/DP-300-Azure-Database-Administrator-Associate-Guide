## 🔍 **Main Idea**

SQL Server databases running on Azure VMs **don’t automatically back up themselves**. You are responsible for setting up **manual or automated backups.** Azure Backup can help, but it’s important to understand the different options and risks.

---

## ✅ **Backup Options for Azure VMs Running SQL Server**

### 1. **Azure VM Backup (Full VM Backup)**

- Azure Backup can back up the **entire VM**, including the OS, SQL Server, databases, and files.
- This is called **application-aware backup**, meaning it coordinates with SQL Server to pause I/O properly (**using VSS**) so you won’t have database corruption.
- ✅ **Good for complete VM restoration** (ex: ransomware recovery).
- ⚠️ **Not always ideal for point-in-time database recovery**. It’s slower if you only want to restore a single database.

If VM snapshot backups cause issues, you may need to add a registry key on the VM:

```
[HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
"USEVSSCOPYBACKUP"="TRUE"
```

---

### 2. **SQL Server Native Backups to Disk or Network Share**

- Backups can be written to:

  - **Local disk** (avoid temporary disks that are cleared on shutdown).
  - **Azure Files** (a network file share in Azure).
  - Any other mounted network share.

- 🔔 Best practice: Copy backups to another location to avoid single points of failure.

---

### 3. **Backup to Azure Blob Storage (Backup to URL)**

- SQL Server can directly back up to **Azure Blob Storage**, using a storage account.
- Example backup path:

  ```
  https://accountname.blob.core.windows.net/containername/mydatabase.bak
  ```

- Authentication is required, using either:

  - **Storage account name + key** (older method, creates page blobs).
  - **Shared Access Signature (SAS)** token → **Recommended**, creates block blobs (cheaper and more secure).

- Easy to restore directly from blob storage using the **RESTORE FROM URL** command.

Example:

```sql
-- Backup transaction log
BACKUP LOG contoso
TO URL = 'https://myacc.blob.core.windows.net/mycontainer/contoso_log.trn'

-- Restore database
RESTORE DATABASE contoso
FROM URL = 'https://myacc.blob.core.windows.net/mycontainer/contoso_full.bak'
WITH NORECOVERY
```

---

### 4. **Automated SQL Backups (SQL Server IaaS Extension)**

- Azure offers a **SQL Server IaaS Agent Extension**, also called the **SQL Server resource provider**, to automate backups:

  - Schedules full, differential, and log backups.
  - Stores backups in your Azure storage account.
  - Manages backup retention.

- Works only on **Windows VMs with SQL Server.**

⚠️ **Critical Note:** Don’t mix backup methods.

- Example mistake: If you configure both the IaaS Extension and inside the SQL Server itself, you could break the **transaction log chain**, making restores impossible without gaps.
- Pick **only one method** for transaction log backups to keep the restore process reliable.

---

## 🔁 **Restoring a Database**

- If backups are on Azure Blob, you restore them the same way as local disk restores, except you use `FROM URL`.
- Automated backups don't include auto-restores. You must restore databases manually using SQL commands or SSMS.

---

## ⚠️ **Key Takeaways**

| Backup Method                             | Can Restore Single DB? | Best For                                   |
| ----------------------------------------- | ---------------------- | ------------------------------------------ |
| Azure VM Backup (snapshot)                | ❌ No                  | Full VM recovery, ransomware recovery      |
| SQL backups to disk or Azure Files        | ✅ Yes                 | Normal SQL Server backup/restore scenarios |
| SQL backups to Blob (Backup to URL)       | ✅ Yes                 | Cloud-based backup and restore             |
| Automated backups with SQL IaaS Extension | ✅ Yes                 | Automated backup management on Azure       |

---
