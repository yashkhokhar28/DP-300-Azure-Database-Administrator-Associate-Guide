## ✅ **What is Active Geo-Replication?**

Active Geo-Replication is a **business continuity and disaster recovery (BCDR) feature** of **Azure SQL Database** (PaaS).
It allows you to create **secondary, read-only copies** of your database in **other Azure regions**.

- These secondary replicas are **asynchronously** updated from the primary.
- If a major disaster happens in the primary region, you can **fail over** to a secondary region and keep your app running.

---

## 🔍 **How It Works**

- Behind the scenes, Azure uses **Availability Group** technology (similar to SQL Server IaaS solutions) to replicate the data.
- Terminology is similar to Always On Availability Groups:

  - **Primary database** → the main database accepting writes.
  - **Secondary databases** → read-only replicas used for reporting or DR.

---

## ✅ **Key Features**

| Feature                | Details                                                        |
| ---------------------- | -------------------------------------------------------------- |
| Type of replication    | **Asynchronous** (no guarantee of 100% sync)                   |
| Replica type           | **Readable**, used for reporting or analytics                  |
| Failover               | **Manual or programmatic failover possible**                   |
| Use case               | Disaster recovery, reporting, global reads                     |
| Platform               | **Azure SQL Database only**, not Managed Instance              |
| Required service tiers | All replicas **must use the same service tier & compute size** |

---

## 🚨 **Azure SQL Managed Instance Does NOT Support This**

For **Azure SQL Managed Instance**, you use **Auto-failover groups**, not Active Geo-Replication.
You'll learn that in the next unit.

---

## 🔧 **How to Configure Active Geo-Replication:**

1. In the Azure portal, navigate to your **Azure SQL Database**.
2. Under **Data Management**, select **Replicas**.
3. Click **+ Create replica**.
4. Choose a different region and configure the secondary database.
5. Azure will handle the replication setup automatically.

---

## 🔁 **Failover Process**

- You can **manually initiate a failover** from the Azure portal.

  - After failover, the secondary becomes **primary**, and the old primary becomes the secondary.
  - Applications must reconnect to the new primary region.

---

## 🌐 **Cross-Subscription Geo-Replication**

If your disaster recovery region is under a **different Azure subscription**, you can still set up geo-replication:

- This requires **programmatic setup using Azure CLI, PowerShell, or REST API.**
- Useful in scenarios where you separate production and DR environments by subscriptions for security or cost management.

---

## ⚙️ **Performance Considerations**

- Secondary replicas should have the **same compute size** as the primary.
- If your database has a **heavy write workload**, using lower-tier replicas could cause replication lag or failures.

---

### ✔️ **Summary of When to Use Active Geo-Replication:**

- You need **regional disaster recovery** for Azure SQL Database.
- You want **read-only replicas** for reporting in another region.
- You’re using **Azure SQL Database single databases or elastic pools**, not Managed Instance.

---
