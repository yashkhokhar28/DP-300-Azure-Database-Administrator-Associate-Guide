## 🔑 **Two HADR Paths in Azure (IaaS vs. PaaS)**

| Type                                       | Think of it like... | Who’s driving?                                                     | Real-World Analogy                                              |
| ------------------------------------------ | ------------------- | ------------------------------------------------------------------ | --------------------------------------------------------------- |
| **IaaS** (SQL on VMs)                      | Renting a car       | **You drive** — install SQL, patch it, set up clusters, cry at 2AM | Running your own data center in Azure’s parking lot             |
| **PaaS** (Azure SQL DB / Managed Instance) | Uber                | **Azure drives** — you just say where to go                        | Handing the keys to Azure and saying “keep it up and available” |

---

## 🛡️ **IaaS Considerations** (aka: You’re the DBA and the SRE)

> 🧠 You're responsible for **everything inside the VM** — SQL Server config, clustering, backups, patching.

### 🔧 Example: High-Availability Setup

Say you're running a banking app in Azure using SQL Server on VMs. You need:

- **Windows Server Failover Cluster (WSFC)**
- **Always On AG** to sync data between nodes
- **Azure Load Balancer** for the listener
- **Cloud Witness** so quorum isn’t toast if a node dies

> 🔥 Real Talk: You’re doing exactly what you did on-prem, except now your cables are made of JSON (ARM templates).

---

## 🌩️ **PaaS Considerations** (aka: Azure’s Got Your Back)

> 🧠 Azure **automatically handles** HA/DR — you just **toggle features**.

### 🔧 Example: SaaS Startup with a Global Userbase

You’re building a learning platform. You don’t want to manage failovers manually — so you choose:

- **Azure SQL Database**
- Enable **Geo-replication** or **Auto-Failover Groups**
- Done. Azure replicates your DB to another region and auto-fails over if region A explodes.

> 💡 Bonus: No patching. No WSFC. No PowerShell pain. Just HA built into the platform.

---

## 🚨 **Key Differences: IaaS vs PaaS**

| Feature               | IaaS (SQL VM)                          | PaaS (Azure SQL / MI)               |
| --------------------- | -------------------------------------- | ----------------------------------- |
| HA/DR setup           | You build it from scratch              | Azure handles it behind the scenes  |
| Backup responsibility | You manage schedules + storage         | Azure backs it up automatically     |
| Failover              | You script/test failovers yourself     | Azure does it for you (auto/manual) |
| Flexibility           | Ultimate control (also: complexity)    | Limited options, but super stable   |
| Best for              | Legacy apps, custom setups, SSRS, etc. | Modern cloud-first apps             |

---

## 🛠️ **Failover Clustering in Azure: IaaS Mode**

You're setting up WSFC on Azure? Okay, a few cloud-specific curveballs:

### Key Tweaks:

| What              | Azure-Specific Gotcha                                   |
| ----------------- | ------------------------------------------------------- |
| **Witness**       | Use **Cloud Witness** instead of shared disk/file       |
| **Networking**    | No static IPs; use Azure DHCP and set IPs in Azure      |
| **Quorum**        | Needed even more in cloud; single point of truth        |
| **AD Dependency** | Optional now (Workgroup Clusters work from Server 2019) |

> 💡 Example: You deploy a 2-node SQL Server cluster across two Availability Zones in East US, use Cloud Witness in West US for quorum — boom, region-tolerant HA.

---

## 🧩 **SQL Server HADR Options in Azure:**

### 1. **Always On Availability Groups (AG)**

> 🧠 Protects **databases** (not the entire instance)

**Scenario:** Multi-node SQL Server on Azure VMs for a financial ERP system. Need HA across zones + readable secondaries.

- Use WSFC + AG
- Add Azure Load Balancer for listener
- Replicas sync over internal Azure VNet

🔥 Works like on-prem AGs, but with Azure’s network quirks.

---

### 2. **Failover Cluster Instances (FCI)**

> 🧠 Protects **entire SQL Server instance**

**Scenario:** You’re migrating a legacy manufacturing app using SQL logins, SSRS, and DTC.

- Setup shared storage (using S2D or premium Azure disks)
- Setup WSFC
- Use DNN instead of VNN if on Windows Server 2019+

🔥 FCIs help when you can’t break monoliths.

---

### 3. **Distributed Availability Groups**

> 🧠 For **multi-region DR**

**Scenario:** You run a stock trading platform. AG in East US, another AG in West US.

- Use Distributed AG to sync between AGs
- Each AG has its own WSFC + listener + load balancer

🔥 Great for globe-spanning DR without building monstrous single clusters.

---

### 4. **Azure Site Recovery (ASR)**

> 🧠 For **VM-level DR**, not app-aware

**Scenario:** You want basic DR for 10 VMs (SQL + app server + job agent).

- Enable ASR → replicates whole VM to another region
- Supports crash-consistent + app-consistent snapshots

🔥 ASR doesn’t know what SQL is — but it’s fast, cheap, and works for “flip-the-switch” DR.

---

## 💎 PaaS-Only HADR Gems

### ✅ **Active Geo-Replication**

> 🌍 Manually-controlled DR for **Azure SQL Database**

- Async replication to up to 4 secondaries
- Failover is **manual**
- Readable secondaries — good for reports

**Example:** E-commerce app, wants East US primary, West Europe for reporting + DR.

---

### ✅ **Auto-Failover Groups (AFG)**

> 🚀 Fully-managed, auto-failover for **SQL Database + MI**

- Automatic cross-region failover
- Uses **read-write** and **read-only** listener names
- Multiple DBs can be grouped

**Example:** HR SaaS app with databases for payroll, leaves, expenses — all fail over together to another region. Apps keep connecting via the listener — no string change needed.

🔥 Managed Instance? You can’t use Geo-Replication — only AFG.

---

## 🧪 Monitoring and Best Practices

Monitoring HA/DR isn’t optional. It’s like having airbags — they better work when you need them.

### Watch These Like a Hawk:

- Azure Service Health (alerts you about outages)
- Replica lag (T-SQL: `sys.dm_database_replica_states`)
- Failover reasons (Resource Health)
- Backup status (Azure Portal / CLI / SSMS)

---

## 🧠 TL;DR Decision Matrix

| Need                           | Best Option                     |
| ------------------------------ | ------------------------------- |
| Total control, legacy features | SQL Server on IaaS + AG or FCI  |
| Minimal admin, max reliability | PaaS (Azure SQL DB / MI) + AFG  |
| Multi-region readable replicas | Active Geo-Replication (SQL DB) |
| VM-level DR across apps        | Azure Site Recovery             |
| DR for apps with multiple DBs  | Auto-Failover Groups            |
| You like PowerShell too much   | IaaS 😉                         |

---

## 🎯 Final Thought

**HADR in Azure** is like building your dream car — you can either:

- Customize every bolt (IaaS), or
- Let Azure hand you the keys to a Tesla (PaaS).

Pick based on your **team skillset**, **compliance needs**, and **app complexity**. Don’t over-engineer it if you don’t need to. But if you're building for scale or compliance, go full architect mode.

---
