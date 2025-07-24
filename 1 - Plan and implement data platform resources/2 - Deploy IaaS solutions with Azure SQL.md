## ✅ **Deploying SQL Server on Azure Virtual Machines (IaaS)**

---

### 🧩 Why Use SQL Server on Azure VMs?

Think of this like **moving your existing SQL Server computer to the cloud**. You get full control—just like managing your own physical machine—but hosted in Azure.

#### 📌 When it makes sense:

- You're migrating from **on-premises SQL** with minimal changes.
- You need full control of **OS, SQL configuration, and infrastructure**.

---

## 🧠 Key Concepts to Understand

### ⚙️ Planning is Essential:

- **VM Size, storage, and network settings** directly affect performance and cost.
- IaaS gives you freedom to **customize for your workload**, just like you'd do with your office servers.

#### 🎯 Learning Outcomes:

- Understand IaaS SQL basics.
- Learn hybrid setups (mix of cloud and on-prem).
- Explore tuning, security, HA (High Availability), and DR (Disaster Recovery) options.

---

## 🧱 Why Choose IaaS for SQL?

- 🧳 **Legacy Compatibility**: Still using older SQL versions? IaaS supports them.
- 🧩 **Full SQL Services**: Run SSIS, SSAS, SSRS together in one VM.
- 🛠️ **Custom Features**: Use CLR, Active Directory auth, or cross-db queries not allowed in PaaS.

---

### 📊 Control Comparison: SaaS vs PaaS vs IaaS

| Feature                   | SaaS | PaaS (Azure SQL DB) | IaaS (SQL VM) |
| ------------------------- | ---- | ------------------- | ------------- |
| SQL Config Customization  | ❌   | Partial             | ✅            |
| OS Patch Control          | ❌   | ❌                  | ✅            |
| Network & Storage Control | ❌   | ❌                  | ✅            |

🔧 **IaaS gives full flexibility**—Microsoft manages hardware, **you manage everything else**.

---

### 🧩 SQL IaaS Agent Extension

Comes with Marketplace VMs to help you:

- Automate **patching and backups**
- Use **Key Vault** + **Defender for Cloud**
- Monitor disks and SQL health
- Apply **best practices**

---

## 💰 Licensing Options

1. **Pay-as-you-go** – license is included in VM cost.
2. **BYOL (Bring Your Own License)** – use your current license with Software Assurance.
3. **Azure Hybrid Benefit** – apply your existing Windows/SQL licenses to save.
4. **Reserved Instances** – commit 1-3 years for discounts.

---

## 💻 VM Types for SQL Workloads

| Series       | Best For                                   |
| ------------ | ------------------------------------------ |
| General      | Dev/test, small databases                  |
| Compute-Opt. | High CPU jobs like app servers             |
| Memory-Opt.  | Large memory SQL workloads (up to 4TB RAM) |
| Storage-Opt. | High-speed storage use                     |
| GPU          | ML, graphics rendering                     |
| HPC/FPGA     | Specialized compute tasks                  |

🛒 **Azure Marketplace** simplifies VM creation using pre-configured templates (e.g., SQL 2022 + Windows).

---

## 🔒 SQL Config During Deployment

Set:

- SQL auth settings
- Instance configs
- Firewall access
- Storage type + performance options

🔁 Use **ARM templates** to automate deployments instead of clicking manually in the Portal.

---

## 🌐 Hybrid Deployments (Mix of On-Prem + Azure)

💡 **Why Use Hybrid?**

- Gradual shift to the cloud
- Cost-effective backups
- Resilient disaster recovery

---

### 🔧 Common Hybrid Setups

1. **Disaster Recovery (DR):**

   - Keep your live server on-prem.
   - Spin up a **failover VM in Azure** during disasters.

2. **Cloud Backups:**

   - Send backups to **Azure Blob Storage**.
   - Restore quickly during failures or drills.

3. **Azure Arc-enabled SQL:**

   - Manage all your SQL Servers (on-prem, cloud, multi-cloud) from **one Azure dashboard**.
   - Includes config checks, inventory, and security scans.

---

### 🔐 Hybrid Networking & Security

- Ensure **AD/DNS** work in both on-prem and Azure.
- Use:

  🔄 **Site-to-Site VPN:** Cheaper, but higher latency.
  🚀 **ExpressRoute:** Faster, private connection (expensive).

---

## 📦 Performance & Security in SQL on Azure VM

### 💽 Storage Tips

| Disk Type      | Use Case               | Max IOPS | Max Speed   |
| -------------- | ---------------------- | -------- | ----------- |
| Ultra Disk     | Critical SQL workloads | 160K     | 10,000 MB/s |
| Premium SSD v2 | High-performance DBs   | 80K      | 1,200 MB/s  |
| Premium SSD    | Most SQL DBs           | 20K      | 900 MB/s    |
| Standard SSD   | Small/light workloads  | 6K       | 750 MB/s    |
| Standard HDD   | Backups                | 2K       | 500 MB/s    |

#### 💾 Disk Setup:

- OS Disk → Windows system
- Temp Disk (D:) → temp files only
- Data Disks → place data/logs with **read/write caching as needed**

#### 🔐 Encryption:

- **At-rest encryption** (managed by Azure)
- **BitLocker** or **DM-Crypt** inside VM (Azure Disk Encryption)

---

## 🚀 SQL Performance Optimization

### ✅ Techniques:

1. **Partitioning** – e.g., monthly sales data for faster queries.
2. **Compression**:

   - Row – low CPU cost
   - Page – better compression
   - Columnstore – good for historical data

#### 🔧 Other Best Practices:

- Enable **backup compression**
- Use **instant file initialization**
- Move logs/system DBs off the OS disk
- Disable **autoshrink/autoclose**
- Cap SQL memory use
- Enable **Query Store** for tracking query plans
- Use **DBCC CHECKDB** regularly

---

## 🔁 High Availability & Disaster Recovery (HADR)

### 🌍 Azure HA Options:

| Feature            | Availability Goal |
| ------------------ | ----------------- |
| Availability Zones | \~99.99%          |
| Availability Sets  | \~99.95%          |

- Zones span datacenters – protect from total site failure.
- Sets spread VMs across hardware racks – protects from hardware issues.

---

### 🛡️ SQL HADR Options

#### 1. **Always On Availability Groups (AG)**

- Replicates DBs to multiple SQL VMs
- **Up to 9 replicas**
- Supports **cross-region DR**
- Use sync (fast) + async (for remote sites)

> ✅ Great for modern apps needing DB-level failover.

#### 2. **SQL Failover Cluster Instance (FCI)**

- Protects entire SQL instance
- Requires shared storage
- Best for **legacy systems**

---

## 🌪️ DR Options Overview

| Option                    | Type                 | Benefit                              |
| ------------------------- | -------------------- | ------------------------------------ |
| SQL Backups to Blob       | DB backup            | Low-cost, geo-redundant              |
| Azure Backup for SQL      | Agent-based          | Long-term retention, auto management |
| Azure Site Recovery (ASR) | VM-level replication | Full failover of whole server        |
| Always On AG (Async)      | DB replication       | Seamless regional failover           |

---

## 🧠 Final Takeaways

- Use **IaaS for control + compatibility**
- Use **Always On AG** + **Availability Zones** for uptime
- Add **backups + ASR** for robust disaster recovery
- Use **Arc** for centralized management—even on-prem
- Follow performance and security best practices for smooth operation

---

### 📘 Real-World Example

A **financial firm** with strict compliance rules keeps production SQL Server **on-prem**, but uses **Azure VMs for disaster recovery** and **Azure Blob for backups**. They apply policies using **Azure Arc** and monitor security via **Defender for SQL**—blending control with cloud flexibility.

---
