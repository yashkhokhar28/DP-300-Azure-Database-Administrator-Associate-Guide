## ✅ **What is a Distributed Availability Group (AG)?**

A **Distributed Availability Group (Distributed AG)** is an **Always On** feature in SQL Server that allows **data replication between two separate availability groups**, possibly running in **different locations** like:

- On-premises SQL Server
- SQL Server running on Azure Virtual Machines (VMs)

### ✔ Purpose in Migration

- Helps migrate databases **with very little downtime**.
- Continuously replicates data from **on-premises SQL Server → Azure SQL Server on VMs**.
- Supports both:

  - **Standalone SQL Server instances**
  - Existing **Always On availability groups (AGs)**.

---

## 🔍 **When to Use Distributed AG for Migration**

| Scenario                                            | Need Distributed AG? | Extra Requirements                                |
| --------------------------------------------------- | -------------------- | ------------------------------------------------- |
| Migrating from a **standalone SQL Server instance** | ✅ Yes               | No need for Windows Failover Cluster or Listener  |
| Migrating an **existing availability group (AG)**   | ✅ Yes               | Requires Windows Failover Cluster and AG Listener |

---

## ⚙️ **How Does Distributed AG Work in Migration?**

- On the **on-premises SQL Server**, configure a distributed availability group.
- Add the **Azure SQL Server VM as a replica** to the distributed AG.
- Continuous replication happens while the **primary server stays online**, minimizing downtime.
- When ready, perform a **failover** to the Azure VM.

---

## 🔧 **How to Configure Distributed AG**

1. **On-premises:** Already have a SQL Server instance (standalone or AG).
2. **Azure:** Prepare SQL Server on Azure VM.
3. **Networking:** Ensure **on-premises and Azure VMs are connected** (usually via a VPN or ExpressRoute).
4. **Add the Azure Replica:**

   - If you’re using the **classic deployment model** in Azure: you can use the **Add Azure Replica Wizard**.
   - **Recommended approach:** Azure Resource Manager (ARM) → you'll need to configure the replica manually using **Transact-SQL (T-SQL)**.

---

## ⚠️ **Limitations / Notes**

- The **Add Azure Replica Wizard** only supports Azure’s **classic deployment model**, which is outdated.
- Microsoft recommends using **Azure Resource Manager (ARM)**, so in that case, you'll configure the distributed AG manually using T-SQL.
- Distributed AG doesn't require **shared storage**, making it easier to extend your SQL Server environment to Azure.

---

## 🔑 **Key Takeaways**

- Distributed AG is a great **low-downtime migration method** for SQL Server.
- No need for downtime during most of the data copy process; only the **cutover (failover) step causes downtime.**
- Works **without clustering** when migrating standalone SQL Servers.
- For **existing Always On AGs**, clustering is required on both source and target.
- Networking between on-premises and Azure is critical for replication.

---
