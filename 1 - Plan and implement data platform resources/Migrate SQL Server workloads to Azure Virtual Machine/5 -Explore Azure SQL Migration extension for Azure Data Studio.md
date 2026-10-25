> ⚠️ **Retired tool (Oct 2026 note):** Azure Data Studio and its Azure SQL migration extension were retired on **28 Feb 2026**. The migrations it ran are now done with **Azure DMS** (portal / PowerShell / CLI), **SQL Server migration in Azure Arc**, or the **SSMS migration component**. See [6 - [NEW] Troubleshoot a migration](<../Migrate SQL Server workloads to Azure SQL/6 - [NEW] Troubleshoot a migration.md>).

## 🎯 **What Is Azure SQL Migration Extension for Azure Data Studio?**

- It is an **extension** for **Azure Data Studio** (a database management tool) that helps you migrate SQL Server databases to **Azure SQL** platforms.
- Behind the scenes, it uses **Azure Database Migration Service (DMS)** to:

  - Handle the migration process.
  - Track migration progress.

---

## 🔁 **Two Migration Modes:**

| Migration Mode | What Happens                                                                                                                                                    | Downtime?                                     |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| **Online**     | The source SQL Server **keeps working normally** (read/write) while backups are restored in the background on the Azure VM. A final cutover happens when ready. | Very short downtime (**only during cutover**) |
| **Offline**    | You **stop all writes** on the source database during the migration. Backups are restored to Azure VM.                                                          | Downtime lasts **throughout the migration**   |

> ⚠️ **Important:**
> You cannot overwrite an existing database on the target SQL Server in Azure VM. You must either:

- Delete the database first, or
- Use a different database name.

---

## 📦 **How Does the Extension Use Backups?**

- This tool **does NOT take backups itself.**
- You must **already have your database backups ready.**
- **Tip:** Use **compressed backups** to speed up the migration.

---

## 🗂️ **Where Should You Store the Backups?**

Two options:

### 1. **Network Share**

- Your backups are on an on-premises file share (e.g., shared folder on your network).
- **You need a self-hosted integration runtime** (a small agent software) on a server that:

  - Can access your backups.
  - Connects securely to Azure Database Migration Service.

🔑 **Setup Details:**

- During migration setup, Azure provides:

  - A download link for the integration runtime.
  - Authentication keys for secure communication.

- For **better performance**, you can:

  - Run **up to 10 concurrent migrations** with one integration runtime.
  - Install multiple integration runtimes to handle bigger migrations.
  - Turn on **auto-update** for security and feature updates.

> ⚠️ **Note:**
> You can’t reuse an existing integration runtime created for **Azure Data Factory**. It must be created for migration.

---

### 2. **Azure Blob Storage Container**

- Upload your backup files to Azure Blob Storage.
- Keep backup files for different databases in **separate folders** to avoid confusion.
- Folder structure supported:

  - Root folder (`/`)
  - Subfolders one level deep (`/DatabaseBackups/YourDatabaseName/`)

---

## ✅ **Summary**

- Azure SQL Migration extension **simplifies the process** of moving databases to SQL Server on Azure VMs.
- You choose **online** or **offline** migration modes depending on your downtime tolerance.
- Backups can come from a **network share** (using integration runtime) or from **Azure Blob Storage**.
- This is a powerful tool for admins who are familiar with Azure Data Studio.

---
