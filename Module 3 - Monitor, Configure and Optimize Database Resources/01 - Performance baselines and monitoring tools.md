# Performance baselines and monitoring tools

> **Exam skills:** Prepare an operational performance baseline, Determine sources for performance metrics, Interpret performance metrics, Configure and monitor activity and performance
> **Reading time:** ~11 min

## The big idea
You cannot say "the database is slow" unless you know what "normal" looks like. So first you record normal numbers (a **baseline**). Then you watch the same numbers every day with the right tools. When a number moves far from normal, you get an alert and you start looking.

Think of a doctor. Your normal heart rate is 70. If it is 140 while you sit still, something is wrong. The doctor only knows this because 70 was written down earlier.

## What is a baseline?
- A **baseline** is a record of your normal performance numbers (CPU, memory, disk, queries per second) over a normal period.
- Later you compare new numbers against it.
- Without a baseline you cannot tell a **normal busy peak** from a **real problem**.

**Example:** A school app normally uses 30–40% CPU during the day. One Monday it shows 95%. Because you have a baseline, you know this is not normal and you start to investigate.

## Look at SQL Server and the OS together
On **SQL Server on an Azure VM** (a virtual machine where you install and run SQL Server yourself), a problem can come from two places:
- **Inside SQL Server**, for example a bad query.
- **Outside SQL Server**, in the operating system (OS) or hardware, for example a slow disk.

So you collect both kinds of numbers and look at them side by side.
- On **Windows**, use **Performance Monitor (Perfmon)** (the built-in Windows tool that records counters).
- On **Linux**, tools like **InfluxDB**, **collectd** and **Grafana** do the same job.

**Example:** A DMV (a built-in system view, explained in lesson 6) shows slow reads from a data file. Perfmon also shows slow reads on the disk. So the disk is slow, not the query. If Perfmon showed a fast disk, you would look at the query instead.

## Key Perfmon counters for a baseline
| Counter | What it shows | What to look for |
|---------|---------------|------------------|
| `Processor(_Total)\% Processor Time` | Total CPU use | High CPU = heavy queries or a VM that is too small |
| `Paging File(_Total)\% Usage` | Use of the page file (disk used as extra memory) | Memory pressure |
| `PhysicalDisk(_Total)\Avg. Disk sec/Read` and `/Write` | Disk delay (latency) per read or write | Normally under 20 ms; under 10 ms on Premium storage |
| `System\Processor Queue Length` | Tasks waiting for a CPU | Above 0 means CPU pressure |
| `SQLServer:Buffer Manager\Page life expectancy` | How long data pages stay in SQL Server memory | A sudden drop = memory pressure or bad queries |
| `SQLServer:SQL Statistics\Batch Requests/sec` | Queries run per second | How busy the server is |
| `SQLServer:SQL Statistics\SQL Compilations/sec` and `Re-Compilations/sec` | How often SQL Server builds query plans | Very high values = query or memory issues |

Also record **wait statistics** (why queries are waiting) with `sys.dm_os_wait_stats`. Lesson 8 covers waits in detail.

## Where performance numbers come from (metric sources)
Different tools see different layers. This is a common exam theme.

| Source | What it sees | Notes |
|--------|--------------|-------|
| **Azure Monitor** (Azure's built-in monitoring service) | Host-level VM numbers: CPU, disk I/O, network | Does **not** show SQL Server internal numbers |
| **VM Insights / enhanced monitoring** | Guest OS numbers: free memory, disk latency, disk space | Installs the **Azure Monitor agent**; data goes to a Log Analytics workspace |
| **Log Analytics workspace** | Stores logs and metrics | You query it with **KQL** (Kusto Query Language) |
| **SQL IaaS Agent extension** | SQL-specific management for a VM in the portal | Marketplace SQL images register automatically; self-installed SQL Server 2016+ is usually auto-registered, otherwise register by hand |
| **Perfmon** | OS and SQL Server counters inside the VM | Can send its data to Azure Monitor |
| **DMVs** | What is happening inside SQL Server right now | Works on all Azure SQL types |
| **Query Store / Query Performance Insight** | Query history and top queries | See below and lesson 7 |

Note: The SQL IaaS Agent extension can also do automated patching, but this is moving to **Azure Update Manager**. The extension's Automated Patching retires 17 Sep 2027.

## Azure Monitor metrics and alerts
- Azure Monitor collects numbers from Azure resources at regular times.
- It keeps metric data for **93 days**. To keep it longer, send it to **Azure Storage** (or a Log Analytics workspace).
- You can draw charts and create **alert rules**.

**Alert rule parts:**
- **Scope** – which resources to watch. For VMs this can be a list of VMs, all VMs in resource groups, or all VMs in a subscription (always in **one Azure region**).
- **Condition** – a metric, an operator (`>`, `<`) and an aggregation (average, max, min, total, count).
- **Threshold** – two types:

| Threshold type | How it works | Good for |
|----------------|--------------|----------|
| **Static** | You set the number, for example CPU > 80% | Simple, fixed limits |
| **Dynamic** | Azure learns from history and sets the limit itself | Workloads with daily or seasonal patterns |

**Action group** – what happens when the alert fires:
- **Notify:** email, SMS, push notification, voice call, email an Azure Resource Manager role.
- **Automate:** Azure Automation runbook, Azure Function, Logic App, ITSM, webhook, secure webhook.
- With no action group, the alert is only logged.

**Example:** An online shop sets a dynamic CPU alert. On Black Friday CPU rises, but Azure knows this pattern, so no false alarm. A strange spike at 3 a.m. does fire the alert, and a runbook emails the on-call admin.

## Key metrics for Azure SQL Database and Managed Instance
In PaaS (Azure runs the server for you) you cannot use Perfmon. You use the portal metrics and DMVs.
- `sys.dm_db_resource_stats` – CPU, data I/O, log I/O and memory for **one Azure SQL database**. One row every **15 seconds**, about **1 hour** kept.
- `sys.server_resource_stats` – resource use for a **Managed Instance**. About **14 days** kept.
- `sys.dm_user_db_resource_governance` – the limits of your Azure SQL database (what your tier allows).
- `sys.dm_instance_resource_governance` – the same for Managed Instance.

**How to read the numbers:**
- CPU near 100% for a long time → find the top CPU queries (Query Store), tune them, or scale up.
- Data I/O near the limit → missing indexes (too many scans) or a tier that is too small.
- Log I/O near the limit → too many writes; you may see `LOG_RATE_GOVERNOR` waits.

## Query Performance Insight (QPI)
**QPI** is a page in the Azure portal that shows your most expensive queries. It reads its data from **Query Store**.
- Where: Azure SQL database → **Intelligent Performance** → **Query Performance Insight**.
- **Azure SQL Database only** (single and pooled). **Not** Managed Instance.
- Needs **Query Store** turned on (it is on by default in Azure SQL Database).

**Three views:**
| View | What it shows |
|------|---------------|
| **Resource consuming queries** | Top queries by CPU, data I/O or log I/O |
| **Long running queries** | Top queries by duration (default: past 24 hours) |
| **Custom** | Choose metric (current docs: CPU, duration, execution count) and time range (24 h, past week, past month) |

**Limits:**
- Shows only the **top 5–20** queries.
- Shows **totals and averages**, not single runs.
- Does **not** show execution plans. Click a query, note its **Query ID**, and open that query in Query Store to see the plan.
- For deeper analysis, Microsoft points to **database watcher** (lesson 2).

**Example:** A food delivery app is slow on Friday nights. In QPI, "Resource consuming queries" sorted by CPU shows one `SELECT` joining Orders and Customers uses 60% of the CPU. The DBA takes its Query ID to Query Store and finds a table scan in the plan.

## Remember for the exam
- A **baseline** = normal numbers recorded first, so you can spot what is abnormal later.
- **Azure Monitor does not show SQL Server internal metrics** on a VM. Use Perfmon, DMVs or the SQL IaaS Agent extension.
- Azure Monitor keeps metrics **93 days**; archive to Storage for longer.
- **Dynamic thresholds** learn from history; good for seasonal patterns.
- Alert scope for VMs is always inside **one region**.
- **QPI = Azure SQL Database only**, needs Query Store, no plans shown, top 5–20 queries.
- Disk latency should be **under 20 ms** (under 10 ms on Premium).
- `sys.dm_db_resource_stats` = one Azure SQL database, 15-second rows, about 1 hour of history.

## Quick check
1. Your SQL Server VM shows high CPU in Azure Monitor. You need to see the SQL Server counter "Batch Requests/sec". Which tool do you use?
   <details><summary>Answer</summary>Performance Monitor (Perfmon) inside the VM, or DMVs. Azure Monitor does not show SQL Server internal counters.</details>
2. You need to find the top 5 CPU-heavy queries of an Azure SQL Managed Instance in the portal using Query Performance Insight. Will this work?
   <details><summary>Answer</summary>No. QPI works only for Azure SQL Database. On Managed Instance use Query Store or DMVs.</details>
3. Your workload is busy every Monday morning. You want alerts only for unusual CPU, not the normal Monday peak. Which threshold type?
   <details><summary>Answer</summary>A dynamic threshold. It learns the normal pattern from history.</details>
