## ✅ **What is Azure SQL Managed Instance?**

Azure SQL Managed Instance is a **Platform as a Service (PaaS)** offering that gives you a **full SQL Server instance in the cloud**, including features that are **not available in Azure SQL Database**.

✔ It is almost **100% compatible with on-premises SQL Server**, making it easy to migrate existing databases and apps to Azure.

✔ **Microsoft manages:** installation, patching, backups, monitoring, and high availability.

---

## 🔑 **Key Features of SQL Managed Instance:**

| Feature                                | SQL Database | SQL Managed Instance |
| -------------------------------------- | ------------ | -------------------- |
| Full SQL Server Engine Features        | ❌           | ✅                   |
| SQL Agent                              | ❌           | ✅                   |
| Cross-database queries                 | Limited      | ✅                   |
| CLR (Custom .NET code execution)       | ❌           | ✅                   |
| TempDB Access                          | ❌           | ✅                   |
| System Databases (msdb, etc.)          | ❌           | ✅                   |
| Restore from on-premises backups       | ❌           | ✅                   |
| Supports multiple databases per server | ❌           | ✅ (Up to 100; 500 on Next-gen GP) 🔄 |
| Manual Copy-Only Backups               | ❌           | ✅                   |

---

## ⚙️ **Service Tiers Available:**

- **General Purpose:** For standard workloads. Uses storage-level redundancy for high availability.
- **Next-gen General Purpose:** upgrade of General Purpose (Elastic SAN storage). Up to **500 databases**, **32 TB** storage, 3 free IOPS per GB with extra IOPS and memory adjustable by slider. Same baseline price as GP. 🔄
- **Business Critical:** For mission-critical workloads. Provides:

  - In-Memory OLTP
  - Readable secondary replica
  - Faster failover with Always On Availability Groups

---

## 🔗 **Managed Instance Link: Hybrid Replication**

- You can replicate data from **on-premises SQL Server to a Managed Instance**, or use it as a **disaster recovery replica**.
- Uses **Distributed Availability Groups**.
- **Transaction log backups** are still needed to manage log space.

---

## ⚙️ **Instance Pools:**

If you have many **small SQL Server instances**, you can deploy them as a **pool** to save costs and speed up deployment (**\~10 min** per instance).

- All instances share the same underlying virtual machine.
- Easier to manage multiple small instances.
- **General Purpose only**. Allows **2-vCore** instances (only possible in a pool). Max **40 instances** and **500 databases** per pool. 🔄

---

## 🔁 **High Availability (HA) and Disaster Recovery (DR):**

- **99.99% SLA uptime**
- **Built-in HA:** Business Critical uses Always On replicas.
- **Auto-failover groups:**

  - Automatically switch to a secondary Managed Instance in another Azure region during outages.
  - No connection string change needed.

---

## 🔐 **Backups & Restore:**

- **Automatic backups:**

  - Full: Weekly
  - Differential: Every 12 hours
  - Transaction Log: Every 5-10 min

- You can do **manual copy-only backups** to Azure Blob Storage (not local disk).
- Supports **long-term backup retention (LTR)** for up to 10 years.

### ➤ **Restore Limitations:**

- Can’t restore a whole instance, only individual databases.
- Can’t restore over an existing database—you must drop/rename it first.
- Only restores from Azure Blob Storage.
- Backups with **FILESTREAM/FileTable** data can't be restored. Backups with active **In-Memory OLTP** objects can't be restored to a **General Purpose** instance (Business Critical is fine). 🔄
- TDE-encrypted backups require the encryption key or certificate during restore.

---

## ⚠️ **Limitations Compared to On-Premises SQL Server:**

- No local file system access.
- Backups and restores must use URL endpoints (Azure Blob).
- No full control over the underlying OS.

---

## 🆕 **Other Current Features** 🔄

- **Stop/start** (General Purpose only): no compute or license charges while stopped. You still pay for storage. Can be scheduled. 🔄
- **Free offer:** one free General Purpose instance per subscription for **12 months** (4 vCores, 720 vCore hours/month, 64 GB storage, up to 100 databases or 500 with Next-gen GP). 🔄
- **Update policy:** **SQL Server 2022**, **SQL Server 2025** (default for new portal deployments), or **Always-up-to-date**. The policy decides whether you can restore back to boxed SQL Server and use MI link bidirectional failover. 🔄

---

## 🔍 **When to Use SQL Managed Instance?**

- Migrating **complex on-premises apps** using SQL Agent, CLR, or cross-database queries.
- Running multiple databases on one instance.
- Needing **easy migration with minimal code changes**.
- Wanting **PaaS benefits (auto-patching, HA, backups)** but **full SQL Server functionality**.

---
