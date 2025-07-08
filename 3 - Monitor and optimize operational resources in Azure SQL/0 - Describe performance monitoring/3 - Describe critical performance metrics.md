## 🎯 **Main Goal:**

Understand how to:

- **Monitor key performance metrics** in Azure Monitor.
- **Create alerts** that notify admins or trigger automated actions.
- **Identify critical SQL Server performance metrics**.

---

## 🔍 **Key Concepts Explained**

### 1. **Azure Monitor Metrics:**

Azure Monitor collects **performance data** from Azure resources (VMs, databases, etc.) at regular intervals.

- You can **analyze** and **visualize** this data in Azure Monitor.
- **Create alerts** when metrics exceed thresholds.
- Azure only keeps **93 days** of metric data unless you archive it to **Azure Storage**.

---

### 2. **Creating Metric Alerts:**

⚙️ **Steps to Create an Alert:**

- Go to **Azure Monitor** in the Azure portal.
- Click **New Alert Rule**.
- Choose the **scope** (which resources to monitor), e.g.:

  - A single VM
  - All VMs in a resource group
  - All VMs in a subscription

Example:
✔️ Resource: Virtual Machine `SQL2019`
✔️ Metric: `% CPU`
✔️ Condition: CPU usage > 80%

You can use two types of thresholds:

| Threshold Type | What it Does                                                                                                                          |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Static**     | Manually define a threshold, e.g., CPU > 80%.                                                                                         |
| **Dynamic**    | Azure analyzes historical data and automatically adjusts the threshold for abnormal activity. Helpful for seasonal workload patterns. |

Alerts support **Boolean operators** (`>`, `<`, etc.) and **aggregation types** (average, max, min, total, count).

---

### 3. **Action Groups: Automating Responses to Alerts**

Once the alert is triggered, you can define **what happens next** using an **Action Group**.

➡️ **Types of Actions:**

| Category     | Examples                                                                                                                   |
| ------------ | -------------------------------------------------------------------------------------------------------------------------- |
| Notification | Send an **Email**, **SMS**, **push notification**, or voice call.                                                          |
| Automation   | Trigger an **Azure Automation Runbook**, **Azure Function**, **Logic App**, or **webhook** to fix the issue automatically. |

If no action group is configured, the alert just gets logged.

---

### 4. **Reviewing Historical Metrics:**

- Azure Monitor lets you view metrics for the **last 93 days**, or longer if archived.
- You can adjust the time filter to analyze recent or older performance trends.

---

### 5. **SQL Server Critical Metrics to Watch:**

| Metric Type            | How to Access                  | Example                                     |
| ---------------------- | ------------------------------ | ------------------------------------------- |
| **SQL Server metrics** | T-SQL DMV queries (`sys.dm_*`) | Data file latency: `sys.dm_os_volume_stats` |
| **OS-level metrics**   | Performance Monitor (perfmon)  | Disk read/write latency                     |

**Example:**

- SQL DMV → find database-level read/write latency.
- Perfmon → find disk latency at the OS level.

👉 By **combining both**, you can tell if the problem is inside the database (**query optimization needed**) or outside (**disk is slow**).

---

## ✅ **Key Takeaways:**

- Use **Azure Monitor** to track resource health and **trigger alerts**.
- Alerts can notify people or **automatically fix issues** (e.g., scale a VM or restart a service).
- For SQL Server, monitor both **database-level metrics** (using DMVs) and **OS-level metrics** (using perfmon).
- **Dynamic thresholds** are smarter and adjust based on historical performance patterns.

---
