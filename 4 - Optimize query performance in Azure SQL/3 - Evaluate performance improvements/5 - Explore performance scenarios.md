## 🛠️ **Two Key Types of Performance Problems**

| Type        | Meaning                                       | Typical Cause                          | Example Wait Types                                |
| ----------- | --------------------------------------------- | -------------------------------------- | ------------------------------------------------- |
| **Running** | SQL Server is actively using resources        | High CPU usage                         | `SOS_SCHEDULER_YIELD`, `CXPACKET`                 |
| **Waiting** | SQL Server is waiting for something to finish | Locking, I/O slowness, memory shortage | `PAGEIOLATCH_SH`, `LCK_M_X`, `RESOURCE_SEMAPHORE` |

---

## 📊 **Monitoring Resource Usage in Azure SQL**

Instead of on-prem tools like **Windows Performance Monitor**, Azure SQL has built-in monitoring:

- **Azure Monitor / Portal** → View usage graphs and configure alerts
- **DMVs (Dynamic Management Views):**

  - `sys.dm_db_resource_stats`: CPU, Memory, I/O for a **single database**
  - `sys.server_resource_stats`: Resource usage for a **Managed Instance**
  - `sys.dm_user_db_resource_governance`: Shows resource limits/capacity for Azure SQL Database
  - `sys.dm_instance_resource_governance`: Same as above, but for Managed Instance

`sys.dm_db_resource_stats` snapshots every 15 seconds (about 1 hour kept); `sys.server_resource_stats` has one row per 15-second interval but is aggregated/updated in 5–10 minute windows (about 14 days kept). 🔄

---

## 🚀 **If the Issue is Running (High CPU)**

You can check these tools to identify **high CPU queries:**

| Tool                            | Purpose                                            |
| ------------------------------- | -------------------------------------------------- |
| **Query Store**                 | Shows top CPU-consuming queries over time          |
| **sys.dm_exec_requests**        | Shows live running queries and their wait types    |
| **sys.dm_exec_query_stats**     | Find cached queries consuming the most CPU         |
| **sys.dm_exec_procedure_stats** | Similar to above but at **stored procedure** level |

Once identified, optimize those queries or **scale up CPU** if needed.

---

## ⏳ **If the Issue is Waiting on Resources**

This scenario involves queries **waiting on locks, I/O, memory, or other bottlenecks.**

Key tools to troubleshoot waiting:

| Tool                        | What it Shows                                                   |
| --------------------------- | --------------------------------------------------------------- |
| **sys.dm_os_wait_stats**    | Shows the **top wait types** across the server                  |
| **sys.dm_exec_requests**    | Shows the **wait type** for each active query                   |
| **sys.dm_os_waiting_tasks** | Shows what **tasks** are waiting for what resources (real-time) |
| **Query Store Wait Stats**  | Summarizes waits per query plan (less detailed than DMVs)       |

---

## ⚙️ **Special Azure SQL Wait Scenarios**

Azure SQL introduces some cloud-specific waits and limits.

### 🔐 **Log Governance Waits**

Azure limits the **log write rate** to protect shared resources.

| Wait Type                          | Scenario                                              |
| ---------------------------------- | ----------------------------------------------------- |
| `LOG_RATE_GOVERNOR`                | Database hitting log write limits                     |
| `POOL_LOG_RATE_GOVERNOR`           | Elastic Pool hitting log limits                       |
| `INSTANCE_LOG_RATE_GOVERNOR`       | Instance-level log limit (the course writes `INSTANCE_LOG_GOVERNOR` for MI) 🔄 |
| `HADR_THROTTLE_LOG_RATE_SEND_RECV_QUEUE_SIZE` | Premium/Business Critical replica not keeping up (course shortens it to `HADR_THROTTLE_LOG_RATE*`) 🔄 |
| `HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO` | Geo-secondary lagging (e.g. smaller SLO than primary) 🔄 |

---

### 👥 **Worker Limits (Threads)**

- Azure SQL Database limits concurrent worker threads (and sessions) **based on the service tier and compute size**. 🔄
- If too many concurrent sessions:

  - Database rejects new queries (**Database tier**) → Error **10928** ("request limit … reached", which really means the worker limit) 🔄
  - Managed Instance → Shows `THREADPOOL` waits

---

### 🌐 **Business Critical Tier Waits**

Business Critical tiers use **Always On availability groups**. You may see:

| Wait Type                          | Meaning                                    |
| ---------------------------------- | ------------------------------------------ |
| `HADR_SYNC_COMMIT`                 | Waiting for replica to confirm transaction |
| `HADR_DATABASE_FLOW_CONTROL`       | Regulating data sent to replicas           |
| `HADR_THROTTLE_LOG_RATE_SEND_RECV_QUEUE_SIZE` | Throttling replication 🔄        |

Long waits here may point to **replica lag** or **I/O issues.**

---

### 🚀 **Hyperscale-Specific Waits**

Hyperscale adds its own storage architecture with **page servers.**

- Unique wait types: `RBIO_*`
- Enhanced metrics for page reads and log governance.

---

## ✅ **Troubleshooting Workflow Example**

1. Check **overall CPU usage** (Azure portal, resource stats DMVs).
2. If CPU is high → Find the **top running queries**.
3. If CPU is normal → Check **wait stats**.
4. Use wait stats to decide:

   - Tune queries/indexes
   - Fix I/O problems
   - Optimize locking
   - Scale up the database

---

## 🔍 **Key Takeaways**

- Always determine if the problem is **"Running" or "Waiting."**
- Use **Query Store** for query-level troubleshooting.
- Use **DMVs** for deeper, real-time investigation.
- Understand **Azure-specific limits** like log governance and worker threads.

---
