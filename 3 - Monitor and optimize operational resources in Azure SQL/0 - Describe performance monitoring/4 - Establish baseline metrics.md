## 📈 **What Is a Baseline?**

A **baseline** is a **record of normal performance measurements** for your system. It helps you:

- Understand how your system performs **under normal conditions**.
- Detect when something is wrong by comparing new data to this baseline.

➡️ **Example:**
If your CPU usage normally stays between 30%–40%, and one day it spikes to 95%, your baseline helps you recognize that this is **abnormal behavior**.

---

## ⚙️ **Why Are Baselines Important?**

Without a baseline:

- You won’t know whether a performance issue is **normal workload behavior** or a **real problem**.
- Every unusual metric could cause unnecessary concern, or worse, real problems could go unnoticed.

---

## 🔗 **Correlating SQL Server and OS Metrics**

When running **SQL Server on Azure VMs**, it’s important to **monitor both the SQL Server and the underlying OS**, because:

- Performance issues might originate in the **SQL Server** (e.g., bad queries).
- Or they might come from the **OS/hardware** (e.g., slow disk, CPU pressure).

If you use:

- **Linux** → You can use tools like **InfluxDB**, **Collectd**, and **Grafana** for metrics and dashboards.
- **Windows** → You can use **Performance Monitor (Perfmon)** for similar tracking.

These tools help you see both sides and pinpoint the source of performance bottlenecks.

---

## 🔍 **Key Performance Monitor Counters for Baselines**

| **Counter Name**                                                                   | **What It Shows**                                         | **Why It’s Important**                                                    |
| ---------------------------------------------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------- |
| `Processor(_Total)\% Processor Time`                                               | Overall CPU usage                                         | High CPU could mean bad queries or underpowered VM.                       |
| `Paging File(_Total)\% Usage`                                                      | How much your system is using virtual memory (page file)  | Memory pressure may cause SQL to run slower.                              |
| `PhysicalDisk(_Total)\Avg. Disk sec/Read` <br> `Avg. Disk sec/Write`               | Disk read/write latency                                   | Should normally be <20 ms; <10 ms on Premium Storage.                     |
| `System\Processor Queue Length`                                                    | Number of processes waiting for CPU time                  | >0 indicates CPU bottleneck.                                              |
| `SQLServer:Buffer Manager\Page life expectancy`                                    | How long data pages stay in SQL Server memory             | A sudden drop means memory pressure or inefficient queries.               |
| `SQLServer:SQL Statistics\Batch Requests/sec`                                      | Number of SQL queries executed per second                 | Helps you understand server activity levels.                              |
| `SQLServer:SQL Statistics\SQL Compilations/sec` and <br> `SQL Re-Compilations/sec` | Number of times SQL needs to create query execution plans | Frequent compilations/recompilations may point to query or memory issues. |

---

## ⏳ **Monitoring Wait Statistics**

SQL Server tracks **why queries are waiting** to execute through **wait statistics**, which you can view using this DMV:

```sql
SELECT * FROM sys.dm_os_wait_stats;
```

- **Waits** happen when SQL queries pause because they’re waiting for resources like CPU, disk, or locks.
- Analyzing **wait types** helps you pinpoint:

  - Poorly optimized queries
  - Hardware limitations
  - Locking/blocking issues

Example:
If your database shows lots of **PAGEIOLATCH waits**, it could indicate **slow disk reads**.

---

## ✅ **Key Takeaways**

- A baseline helps you define **“normal” system behavior**.
- Correlate SQL Server and OS metrics to see the **full performance picture**.
- Regularly monitor:

  - CPU, memory, and disk from the OS
  - SQL query activity and wait statistics

- Without baselines, it’s harder to tell the difference between **normal peaks** and **real problems**.

---
