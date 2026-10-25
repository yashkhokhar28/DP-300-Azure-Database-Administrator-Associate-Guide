## ✅ **Azure SQL as a PaaS Offering**

**Platform as a Service (PaaS):**

- Microsoft manages **infrastructure**, **OS**, and **SQL Server updates**.
- You only manage your **databases** and **data**.
- Ideal for reducing administration work and improving scalability.

Azure SQL PaaS offerings include:

- **Azure SQL Database** (single database or elastic pool)
- **Azure SQL Managed Instance** (for easier migration from on-premises SQL Server)

---

## 🔍 **Deployment Models in Azure SQL Database**

### 1. **Single Database**

- Each database runs independently.
- Great for isolated workloads with predictable performance.
- Billing and scaling are done **per database**.

### 2. **Elastic Pool**

- Group of databases sharing resources.
- Ideal when databases have **variable workloads** that don’t peak at the same time.
- Cost-effective for SaaS applications.

---

## 💳 **Purchasing Models**

### A. **DTU (Database Transaction Unit) Model**

- Combines CPU, memory, and I/O into a single performance unit.
- Service tiers: Basic, Standard, Premium.
- **Simple but less flexible** for fine-tuning resources.

### B. **vCore Model** _(Recommended)_

- You choose the number of vCores, memory, and storage.
- More control and **better aligned with on-prem SQL Server licensing** (Azure Hybrid Benefit).
- Three service tiers:

| Tier                  | Best for          | Storage         | Resilience          | Max DB Size |
| --------------------- | ----------------- | --------------- | ------------------- | ----------- |
| **General Purpose**   | Regular workloads | Premium Storage | Moderate            | 4 TB        |
| **Business Critical** | High performance  | Local SSD       | High (read replica) | 4 TB        |
| **Hyperscale**        | Huge DBs (recommended default for new OLTP) 🔄 | Decoupled storage + local SSD cache 🔄 | High                | 128 TB 🔄   |

---

## ⚙️ **Serverless Compute Option (General Purpose and Hyperscale tiers)** 🔄

- Scales compute **automatically** based on workload.
- **Pauses when idle**, so you only pay for storage when inactive. Auto-pause delay: **15 min – 7 days** in General Purpose (60 min min. in Hyperscale, where auto-pause is preview); can be disabled (-1). 🔄
- Useful for databases with unpredictable or sporadic workloads.
- Limitations: geo-replication/failover groups, long-term retention, a DNS alias, a sync database (Data Sync) or an elastic jobs **job database** **prevent auto-pause** (auto-scaling still works). 🔄

---

## 🔒 **Backups**

- Fully **automated backups**:

  - Full backup: weekly
  - Differential backup: every 12 or 24 hours (default 12 h in vCore, 24 h in DTU) 🔄
  - Log backups: every 5–10 minutes

- Retention: **1 to 35 days** (default **7**; Basic tier max 7), can be extended to **10 years** with Long-Term Retention (LTR). 🔄
- Backup storage redundancy is **geo-redundant by default**; you can choose zone- or locally-redundant. Hyperscale uses storage **snapshots** instead of full/diff/log backups. 🔄
- Restore options:

  - Restore to **a new database** only (can't overwrite existing databases).
  - Use **Azure Portal, PowerShell, or CLI**.

---

## 🌍 **High Availability and Disaster Recovery**

### **Active Geo-Replication**

- Creates up to **4 secondary read replicas** in different regions.
- Great for **read scaling** and disaster recovery.
- Failover is **manual or application-driven**, requires changing connection strings.

### **Auto-Failover Groups**

- Builds on geo-replication.
- Provides a **single connection endpoint**.
- Failover policy: **Customer managed (recommended)** = you fail over ("manual" in CLI/PowerShell); **Microsoft managed** = automatic, only for widespread regional outages. No connection string change needed either way. 🔄

---

## ✔️ **Summary**

Azure SQL PaaS (Database & Managed Instance) lets you:

- Focus on your data and apps, while Microsoft handles backups, patching, and infrastructure.
- Choose how much control you want over compute and storage (vCore vs. DTU).
- Easily scale up/down or pause databases when needed.
- Improve availability and disaster recovery with geo-replication and failover groups.

---
