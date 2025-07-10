## 🔑 **Key Idea: Two Different Approaches to HADR**

| Approach                               | Description                                                                                                                                                                                                         |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **IaaS (Infrastructure as a Service)** | You manage and configure SQL Server **on virtual machines (VMs)**. Azure provides the infrastructure, but you must set up HA/DR features like **Availability Groups (AG)** or **Failover Clusters (FCI)** yourself. |
| **PaaS (Platform as a Service)**       | Azure fully manages the database engine (**Azure SQL Database** or **SQL Managed Instance**). HA/DR features are built-in, and you enable them through **Azure features**, not server-level configurations.         |

---

## 🛡️ **IaaS Considerations**

When using IaaS for SQL Server:

- You need to configure **Windows Server Failover Clusters (WSFC)** if you're using **Always On Availability Groups (AG)** or **Failover Cluster Instances (FCI)**.
- Azure provides the infrastructure (**VMs, storage, networking**) but you manage:

  - SQL Server installation.
  - Cluster setup.
  - Patching and upgrades.

- **Azure features like Availability Sets or Availability Zones** help improve uptime but don’t replace your SQL Server HA setup.

💡 **Key difference from on-premises:**
Azure uses virtualized hardware and networks, so some hardware-level failure scenarios (like network card failure) are less likely, but other cloud-level issues (like region-level outages) must be planned for.

---

## 🌩️ **PaaS Considerations**

With **PaaS services (Azure SQL Database / Managed Instance):**

- Azure **automatically handles HA/DR**.
- You just enable features like:

  - **Auto-failover groups** (for automatic region failover).
  - **Geo-replication** (for cross-region redundancy).

- RTO and RPO goals are met through Azure’s built-in service-level agreements (**SLA**).
- No need to configure OS, clusters, or backups for HA/DR — Azure manages that.

---

## 🔍 **Summary of Differences**

| Aspect                        | IaaS                                           | PaaS                                    |
| ----------------------------- | ---------------------------------------------- | --------------------------------------- |
| You manage...                 | SQL Server install, OS, HADR features          | Only the database & connection settings |
| HA/DR setup                   | Configure AG/FCI + Azure availability features | Built-in, mostly automatic              |
| Backup/restore responsibility | You manage backups and restores                | Azure manages backups automatically     |
| Typical products              | SQL Server on Azure VM                         | Azure SQL Database / Managed Instance   |

---

## ✅ **Goal of the Module**

By the end of the module, you will:

- Understand how to deploy IaaS database platforms with HA/DR in Azure.
- Know the **differences between on-premises and Azure setups**, especially for failover clusters and AGs.
- Be able to **choose between IaaS and PaaS**, depending on your HA/DR needs.

---

### ▶️ Next Topic

> **Describe failover clusters in Windows Server** — you'll learn how to set up the **failover clustering** that supports SQL Server HA in IaaS.
