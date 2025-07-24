## ✅ **SQL Server on Azure Virtual Machines (VM)**

### 🧠 **What Is It?**

Think of **SQL Server on Azure VM** like running SQL Server on a regular PC or server in your office—but instead, it runs in Microsoft's data center (Azure). It's like renting a computer from Microsoft where you control everything—Windows, SQL version, configurations.

> 📌 **Real Example:** You're a database admin at a retail company. Instead of managing physical servers in 10 cities, you move them to Azure and control them from one place.

---

### 🏁 **Why Use SQL Server on Azure VM?**

Use it when you:

- Need **full control** over SQL Server and the OS.
- Want to use **advanced features** like FileStream, PolyBase, or cross-instance transactions.
- Run apps that need to be **moved together** (like an HR system and its database).
- Need to stick with an older SQL version (like SQL Server 2016).
- Are moving from an on-prem server “as is.”

---

### 📦 **Business Scenario**

You're a Senior DBA in a retail company with many different apps and SQL versions. You need to modernize by moving to Azure. Your job is to:

- Review what's currently running.
- Decide which servers go to Azure VM.
- Migrate with the least downtime.
- Confirm everything works after the move.

---

### 📚 **What You'll Learn**

1. Online vs. Offline migrations
2. Migration tools (Azure Migrate, Backup/Restore, DMA, etc.)
3. What to check after migrating

---

### 🛡️ **Benefits**

- **Full control** over SQL Server settings
- Run **all SQL features** (replication, Always On, FileStream, etc.)
- Store huge databases (up to 256 TB)
- **Easy backup and high availability**
- Compatible with your current SQL setup

---

### 💰 **Licensing Options**

| Model                             | When to Use                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| **Pay-as-you-go**                 | For short-term or testing servers                            |
| **BYOL (Bring Your Own License)** | When you already have SQL licenses (with Software Assurance) |
| **HA/DR License**                 | For free replicas in disaster recovery setup                 |

---

### 🌐 **Connectivity Types**

- **Public:** Accessible via internet
- **Private:** Only inside your company’s network (more secure)
- **Local:** Only accessible from within the VM

---

### 🔐 **Key Management with Azure Key Vault**

- You can use Azure Key Vault to **secure encryption keys** for SQL Server.
- This helps with **compliance** and **data security**.

---

### ⚙️ **VM Sizing Tips**

| Resource     | Recommendation                                              |
| ------------ | ----------------------------------------------------------- |
| VM Size      | Use DS3_v2+ (for Enterprise); DS2_v2+ (for small workloads) |
| Storage      | Use **Premium SSDs**                                        |
| Disks        | Separate data, log, and TempDB on different disks           |
| Disk Caching | Use read caching on data, none on log                       |
| SQL Config   | Enable compression, disable autoshrink                      |

---

## 🔧 **Migration Tools Overview**

| Tool/Method         | Use Case                                 |
| ------------------- | ---------------------------------------- |
| Azure Migrate       | Full VM migration                        |
| DMA                 | Check compatibility and move schema/data |
| Backup & Restore    | Simple, offline method                   |
| Log Shipping        | Ongoing sync with small downtime         |
| Always On AG        | Zero downtime                            |
| Detach/Attach       | Move large DBs manually                  |
| bcp / Import Export | Table-level or smaller migrations        |
| Azure Data Factory  | Move data from many sources              |
| Disk Import Service | Ship physical drives if internet is slow |

> 📌 **Example:** If your store’s CRM database is 1 TB and can't be offline for long, use **Always On AG** for live migration.

---

### 🕒 **Migration Downtime Options**

| Method           | Downtime             |
| ---------------- | -------------------- |
| Always On AG     | None                 |
| DMS (online)     | Few minutes          |
| Backup & Restore | Hours (offline time) |

---

### ✔️ **Key Migration Steps**

1. Analyze your current environment
2. Choose the right tool and downtime model
3. Size your Azure VM correctly
4. Set up networking and security
5. Migrate
6. Post-check: Test performance and functionality

---

## 🧩 **Azure SQL Migration Extension (Azure Data Studio)**

This is a plugin for **Azure Data Studio** that makes migration easier.

- Works with **Azure Database Migration Service (DMS)** in the background
- Supports **online** and **offline** modes
- Doesn’t create backups—it uses **your own**
- Backups can come from:

  - **Network shares** (needs a small agent called Integration Runtime)
  - **Azure Blob Storage**

> 🔄 **Online Migration:** Database stays live, you only switch over at the end.

> 🔒 **Offline Migration:** Database is stopped during the move.

---

### 📦 **Using Azure Blob Storage for Backups**

SQL Server on Azure VM can **directly back up to Blob Storage** using:

```sql
BACKUP LOG mydb TO URL = 'https://...'
```

And restore with:

```sql
RESTORE DATABASE mydb FROM URL = 'https://...'
WITH NORECOVERY
```

> 📌 **Real Example:** Your company exports a 500 GB backup from on-prem and stores it on Blob. Azure VM picks it from there to restore.

---

### 🧰 **Tools for Transfer**

- Use **AzCopy** CLI for fast file transfers to Azure
- Organize folders inside Blob containers:

  - `/FULL`
  - `/DIFF`
  - `/LOG`

---

### 🧠 **Backup Tips**

| Task          | Tip                                       |
| ------------- | ----------------------------------------- |
| Large backups | Use `WITH COMPRESSION`                    |
| >200 GB       | Use striped backups (multiple files/URLs) |

---

## 🧪 **Data Migration Assistant (DMA)**

- Use it to check for problems before migrating.
- Migrates:

  - Schema
  - Data
  - Logins

- Highlights:

  - Deprecated features
  - Compatibility issues

> ⚠️ **Limitation:** It won’t migrate certificate-based logins or the `sa` account.

---

### 🔧 **DMA Custom Settings**

Edit config file to:

- Increase number of parallel DB migrations
- Increase connection timeout

---

### ✅ **Best Practices for DMA**

- Don’t run DMA on production servers
- Use backups for testing
- Encrypt migration traffic
- Run assessments outside business hours

---

## 🔁 **Distributed Availability Groups (AG)**

A special type of Always On setup that lets you connect two availability groups across locations.

- Ideal for migrating SQL Server to Azure VM **with almost zero downtime**
- Works with:

  - Standalone SQL Servers
  - Existing Always On setups

---

### 🛠️ **How It Works**

1. Set up on-prem AG or instance
2. Connect Azure VM to it as a replica
3. Data continuously syncs
4. Do a final failover (cutover)

> 📌 **Real Example:** You're moving a busy POS database from Mumbai datacenter to Azure. You keep syncing using Distributed AG, then switch traffic when ready.

---

### ⚠️ **Things to Know**

- You’ll need **network connectivity** (VPN/ExpressRoute)
- Azure **Resource Manager (ARM)** method is preferred (not classic wizard)
- **No shared storage** needed

---

## 🎯 **Final Takeaways**

- SQL Server on Azure VM = Full control, best for lift-and-shift
- Use Azure Migrate, Backup/Restore, DMS, DMA, and AG depending on your needs
- Plan for downtime—zero, short, or long
- Use Blob storage smartly for performance
- Test everything after migration

---
