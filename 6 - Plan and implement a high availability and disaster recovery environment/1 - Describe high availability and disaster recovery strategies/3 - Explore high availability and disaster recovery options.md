## 🔍 **IaaS vs. PaaS Availability**

| Aspect      | IaaS (Virtual Machine)                                   | PaaS (Azure SQL Database / Managed Instance)     |
| ----------- | -------------------------------------------------------- | ------------------------------------------------ |
| What it is  | You manage the OS and SQL Server                         | Microsoft manages SQL Server & infrastructure    |
| HADR Setup  | You choose, configure, and manage SQL Server HA/DR tools | HADR is built-in; you just enable/configure them |
| Flexibility | Full control, many options                               | Limited customization, easier setup              |

Your **choice affects how you design your availability and disaster recovery.**

---

## ⚙️ **SQL Server HADR Features in IaaS**

### 1. **Always On Failover Cluster Instance (FCI)**

- **Protects:** The **entire SQL Server instance** (all databases, logins, jobs, etc.).
- **Needs:** Windows Server Failover Cluster (WSFC) or Pacemaker (Linux), shared storage, and an **internal load balancer (ILB)** in Azure.
- **Failover:** Restarts SQL Server on another node, disconnecting users temporarily.
- **Storage:** Requires shared storage like Azure Shared Disks or Premium File Share.
- **Used For:** Protecting everything in one instance, with a single database copy shared across nodes.

---

### 2. **Always On Availability Groups (AGs)**

- **Protects:** **Databases only**, not the whole instance.
- **Needs:** WSFC or Pacemaker for clustering. No shared storage (each replica has its own copy of the database).
- **Failover:** Faster than FCI, with minimal downtime.
- **Storage:** More expensive—each replica has a **full copy** of the database.
- **Editions:** Standard Edition allows 1 database and 2 replicas; Enterprise supports multiple databases and up to 9 replicas.
- **Bonus:** Secondary replicas can be used for **read-only queries** and backups.

---

### 3. **Log Shipping**

- **Protects:** Databases only (like AG).
- **How it works:** Periodically backs up the transaction log from the primary database, copies it to a secondary server, and restores it there.
- **Failover:** **Manual failover, no automatic switching.**
- **Simple and cost-effective**, but not real-time.

---

## ➕ **Key Differences Between FCI, AG, and Log Shipping**

| Feature        | FCI                                 | AG                                            | Log Shipping                    |
| -------------- | ----------------------------------- | --------------------------------------------- | ------------------------------- |
| Protects       | Entire SQL Server instance          | Individual databases                          | Individual databases            |
| Shared storage | Required                            | Not needed                                    | Not needed                      |
| Failover type  | Full instance restart               | Database-level switch                         | Manual failover                 |
| Automation     | Automatic failover                  | Automatic failover                            | No automatic failover           |
| Complexity     | More complex (shared storage, WSFC) | Medium (clustering, multiple database copies) | Simple (backup/restore process) |
| Cost           | Medium to high                      | High (multiple copies of data)                | Low                             |

---

## 📌 **Important Notes**

- For **IaaS solutions**, you are responsible for configuring clusters (Windows Server Failover Cluster or Pacemaker for Linux).
- In **Azure**, you need an **Internal Load Balancer (ILB)** to ensure proper connection redirection during failover in FCI setups.
- For FCI, **Active Directory and DNS** are mandatory.

---

## 🌩️ **PaaS Simplifies Things**

- In **Azure SQL Database / Managed Instance**, Microsoft **automatically handles failover, backups, and high availability.**
- You have **less control but less maintenance burden.**

---

## ✔️ Summary Takeaways

- **FCI:** Protects the whole SQL Server instance. Good for keeping everything (logins, jobs, etc.) together. Needs shared storage and cluster setup.
- **AG:** Protects databases, not the whole instance. Quicker failovers, no shared storage. Higher storage costs due to multiple copies of data.
- **Log Shipping:** Basic DR feature using transaction logs. Manual failover. Simple and reliable for secondary standby databases.

---
