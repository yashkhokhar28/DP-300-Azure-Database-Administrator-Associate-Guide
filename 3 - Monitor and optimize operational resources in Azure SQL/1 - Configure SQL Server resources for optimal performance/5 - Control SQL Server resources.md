## ✅ **What is Resource Governor?**

Resource Governor is a **SQL Server feature (since 2008)** that lets you control **how much CPU, memory, and I/O** different groups of sessions can use on the server.

➡️ It’s mainly useful when **multiple applications** or **users** share the same SQL Server instance, and you want to ensure one heavy workload doesn't slow down others.

---

## 🔧 **How Does Resource Governor Work?**

### 1. **Resource Pools**

- Resource Pools are the **physical resources** (CPU, memory, I/O) available on the server.
- SQL Server has:

  - **Internal Pool**: Reserved for critical system operations (**can’t be limited**).
  - **Default Pool**: Where most sessions go unless otherwise configured.
  - **Custom Pools**: You can create your own for specific workloads.

➡️ **Settings you can control for each pool:**

| Setting                 | Meaning                                           |
| ----------------------- | ------------------------------------------------- |
| Min/Max CPU %           | Controls CPU usage during contention.             |
| Cap of CPU %            | **Hard** upper limit of CPU usage — enforced even when there's no contention. 🔄 |
| Min/Max Memory %        | Memory limits.                                    |
| NUMA Node Affinity      | Bind workloads to specific processors (advanced). |
| Min/Max IOPS per volume | Controls I/O operations per storage volume.       |

⚠️ Changes to pools only affect **new sessions**, except for external scripts in Machine Learning.

---

### 2. **Workload Groups**

- These are **groups of sessions** classified together.
- Each group is **linked to one resource pool**, but a pool can have **many groups**.
- SQL Server includes:

  - **Default Group**: For all sessions unless classified differently.
  - **Internal Group**: For system operations.

---

### 3. **Classifier Function**

- A **T-SQL user-defined function** that **decides which session goes into which workload group**.
- Runs **when a user connects**.
- Example: assign different users to different groups:

```sql
CREATE FUNCTION dbo.RGClassifier()
RETURNS SYSNAME
WITH SCHEMABINDING
AS
BEGIN
DECLARE @WorkloadGroup AS SYSNAME
IF(SUSER_NAME() = 'ReportUser')
    SET @WorkloadGroup = 'ReportServerGroup'
ELSE IF (SUSER_NAME() = 'PrimaryUser')
    SET @WorkloadGroup = 'PrimaryServerGroup'
ELSE
    SET @WorkloadGroup = 'default'
RETURN @WorkloadGroup
END
```

➡️ This is a **simple example based on username**, but you can make it more complex (e.g., based on application name, hostname, etc.).
⚠️ Keep the function **efficient**, as it runs on every connection.

---

## 📈 **When to Use Resource Governor**

- **Multi-application servers:** Keep one app’s heavy workload from starving others.
- **Multi-tenant databases:** Different customers share a server.
- **Control maintenance tasks:** Limit CPU for index rebuilds or backups during business hours.

---

## 🔍 **Example Use Case**

| Scenario                            | Resource Governor Action                                |
| ----------------------------------- | ------------------------------------------------------- |
| Reporting job runs during peak time | Limit its CPU to avoid slowing down online transactions |
| Index rebuild overnight             | Allow max resources but limit during daytime            |
| Shared instance for 3 apps          | Give each app a dedicated resource pool                 |

---

### ✔ Key Points

- Works in **SQL Server** and **Azure SQL Managed Instance**.
- Not available in **Azure SQL Database** (because the platform automatically manages resources).
- On **Managed Instance**, Resource Governor config lives in `master` and **replicates to secondary replicas** (HA and geo); in SQL Server AGs you must configure each replica. 🔄
- **SQL Server 2025**: Resource Governor is now also in **Standard** edition (previously Enterprise/Developer only) and can cap **tempdb space** per workload group (tempdb space resource governance). 🔄
- Helps prevent one session or workload from consuming all the server’s resources.

---
