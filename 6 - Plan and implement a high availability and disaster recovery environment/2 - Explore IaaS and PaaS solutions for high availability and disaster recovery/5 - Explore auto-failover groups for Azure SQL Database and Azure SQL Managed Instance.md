## ✅ **What Are Auto-Failover Groups?**

**Failover groups** (formerly called auto-failover groups) are a **built-in cross-region disaster recovery (DR) solution** in **Azure SQL Database** and **Azure SQL Managed Instance**. 🔄
They allow you to:

- Automatically replicate your databases to another region (**geo-replication**).
- Automatically or manually fail over your databases if something goes wrong in the primary region.

---

## 🔍 **Key Features of Auto-Failover Groups**

| Feature              | Description                                                                                                |
| -------------------- | ---------------------------------------------------------------------------------------------------------- |
| Supported in         | **Azure SQL Database & Azure SQL Managed Instance**                                                        |
| Failover             | Automatic or manual failover to a secondary region                                                         |
| Listener support     | Provides a **read-write listener** and a **read-only listener**                                            |
| Transparent failover | Applications connect using a **listener name** (DNS-based). No need to manually update connection strings. |
| Scope                | SQL DB: some or all DBs on a server, multiple groups per server. SQL MI: **all user DBs**, **one** group per instance, system DBs not replicated 🔄 |
| Unique name          | The failover group has a **unique name** in the `*.database.windows.net` domain                            |

---

## 🎯 **How It Works**

- The **primary server** hosts the read-write copy of your databases.
- The **secondary server** (in another region) hosts a **read-only copy**, which stays in sync asynchronously.
- In the event of a failure in the primary region, Azure:

  - Fails over the databases to the secondary.
  - **Automatically updates DNS** so that applications reconnect to the new primary without needing manual changes.

---

## ⚙️ **Failover Policies**

| Policy Type | Default Behavior | Customization |
| ----------- | ---------------- | ------------- |
| Read-write: **Customer managed** (`Manual`) | **Recommended**. You decide when to fail over (planned or forced) 🔄 | Use it to test and run your DR plan 🔄 |
| Read-write: **Microsoft managed** (`Automatic`) | Microsoft forces failover of **all** such groups in the region, only for a wide regional outage, after the grace period (min **1 hour**) 🔄 | Never for a single group. Don't rely on it 🔄 |
| Read-only | Read-only listener failover is **disabled by default**: read-only traffic isn't sent to the primary while the secondary is offline (protects primary performance) 🔄 | You can enable it so read-only sessions fall back to the primary 🔄 |

---

## ⚠️ **Data Loss Considerations**

- Failover is asynchronous, so there is a chance of **data loss during unplanned failover.**
- With the **Microsoft managed** policy, **`GracePeriodWithDataLossHours`** defines how long Azure waits after the outage before it starts a **forced** failover (possible data loss). 🔄

  - Default and minimum: **1 hour** 🔄
  - Set higher if your workload can’t afford much data loss.

---

## 🔁 **Difference from Active Geo-Replication**

| Feature                                   | Geo-Replication     | Auto-Failover Groups     |
| ----------------------------------------- | ------------------- | ------------------------ |
| Automatic failover                        | No                  | Only with the Microsoft-managed policy (customer managed is recommended) 🔄 |
| Supports multiple databases in a group    | No                  | Yes                      |
| Connection string changes after failover? | Yes (manual update) | No (handled by listener) |
| SQL Managed Instance support              | No                  | Yes                      |
| Can replicate to the same region?         | Yes                 | No (only cross-region)   |
| Supports multiple secondary replicas      | Yes (up to 4) 🔄    | One secondary (multiple secondaries are in **preview** for SQL DB) 🔄 |
| Supports read-scale                       | Yes                 | Yes                      |

---

## 🛠️ **Setup Process Summary**

1. Create a **failover group** in the Azure portal, CLI, or PowerShell. 🔄
2. Assign one or more databases to the failover group.
3. Azure automatically creates and seeds the secondary databases.
4. Configure your applications to connect via the failover group’s **listener name**.

---

## 🌟 **Benefits of Auto-Failover Groups**

- Fully managed HA/DR for Azure SQL Database and Managed Instance.
- **No need to update connection strings** after failover.
- Supports failover of multiple databases together.
- Simplifies DR configuration compared to traditional IaaS SQL Server.
- **License-free DR replica**: if the secondary is used **only for DR** (no read workload), designate it as a **standby** replica. You don't pay SQL Server licensing for its vCores (compute and storage are still billed). Available for SQL DB geo-secondaries and SQL MI failover groups. 🔄
- **MI requirements**: both instances in the **same DNS zone** (`DNSZonePartner`), connected by **global VNet peering** (recommended), VPN, or ExpressRoute. 🔄

---
