# Extended Events, database watcher and Intelligent Insights

> **Exam skills:** Monitor by using Extended Events, Monitor by using database watcher, Monitor by using Intelligent Insights
> **Reading time:** ~11 min

## The big idea
These are three deeper monitoring tools.
- **Extended Events** is a light recorder you switch on to catch exact events, like deadlocks.
- **Database watcher** is a managed dashboard that collects numbers from many Azure SQL databases in near real time.
- **Intelligent Insights** is built-in AI that notices when performance gets worse and writes down the likely cause.

Think of a shop. Extended Events is a security camera you point at one door. Database watcher is the control room with screens for every shop in the chain. Intelligent Insights is a smart guard who writes a report: "Sales slowed today because the till was broken."

## Extended Events (XE)
**Extended Events** is the light, modern tracing system in SQL Server, Azure SQL Database and Managed Instance. It replaces the old **SQL Profiler**.
- Low overhead (it slows the server very little), especially with good filters.
- Can watch almost anything: queries, locks, deadlocks, I/O, errors.

**Common uses:**
- Catch **deadlocks** and blocking.
- Find **long-running queries**.
- Watch **DDL** (changes to tables and other objects).
- Log missing statistics.
- Track memory pressure or slow disk I/O.

**Example:** A school app gets a deadlock error twice a day. The DBA starts an XE session that records the deadlock report. Next day the report shows which two procedures fight over the same rows.

### The four parts of a session
| Part | Meaning | Example |
|------|---------|---------|
| **Event** | The thing to catch | `sql_statement_completed`, `lock_acquired` |
| **Action** | Extra data to add to each event | Session ID, SQL text |
| **Predicate (filter)** | Which events to keep | Only one database, only queries over 5 seconds |
| **Target** | Where to store the results | A file or memory |

### Targets
| Target | What it does | Sync or async | Where |
|--------|--------------|---------------|-------|
| **event_file** | Saves events to a file for later | Asynchronous | All. In Azure SQL DB and MI the file goes to **Azure Blob Storage** |
| **ring_buffer** | Keeps recent events in memory; lost when the session stops | Asynchronous | All |
| **event_counter** | Just counts each event | Synchronous | All |
| **histogram** | Counts events grouped by a field value | Synchronous | All |
| **pair_matching** | Finds paired events that did not match (for example lock taken but not released) | Asynchronous | **SQL Server only** |
| **ETW** (Event Tracing for Windows) | Joins SQL events with Windows OS traces | Synchronous | **SQL Server only** |

- Use **event_file** for long sessions you want to keep.
- Use **ring_buffer** for quick live checks.
- In **Azure SQL Database and MI**, the `event_file` target needs a **credential** to the storage account (managed identity or SAS token).

### Server vs database scope
- **SQL Server and Managed Instance:** `CREATE EVENT SESSION ... ON SERVER`
- **Azure SQL Database:** sessions are **database-scoped**: `CREATE EVENT SESSION ... ON DATABASE`

```sql
CREATE EVENT SESSION LongQueries ON DATABASE   -- Azure SQL Database
ADD EVENT sqlserver.sql_statement_completed
    (ACTION (sqlserver.sql_text, sqlserver.session_id)
     WHERE duration > 5000000)                 -- microseconds = 5 seconds
ADD TARGET package0.ring_buffer;
ALTER EVENT SESSION LongQueries ON DATABASE STATE = START;
```

In SSMS: **Management → Extended Events → Sessions → New Session** (templates such as **Locks and Blocks** and **Query Execution**).

**Best practice:** always add filters, catch only the events you need, and stop sessions you no longer use.

## Database watcher
**Database watcher** is Azure's managed monitoring service for **Azure SQL Database** and **Azure SQL Managed Instance**. It is in **preview**.
- It collects monitoring data (from DMVs) **within seconds** (near real time).
- It shows ready-made **dashboards** in the Azure portal.
- It replaces **SQL Insights**, which was retired on 31 Dec 2024.

**What it can watch:** performance counters (CPU, I/O, waits), query performance, deadlocks and blocking, elastic pool use, configuration changes, Managed Instance health.

**Supported targets:** all Azure SQL Database and MI service tiers and purchasing models (DTU, vCore, serverless, Hyperscale, elastic pools), plus readable, geo and named replicas. **Not SQL Server on Azure VMs.**

**Where the data is stored (data store):**
| Data store | Notes |
|-----------|-------|
| **Azure Data Explorer (ADX) cluster** | Good for large amounts of time-series data; query with KQL |
| **Free ADX cluster** | For small workloads or testing |
| **Real-Time Intelligence in Microsoft Fabric** | Same engine as ADX, as a SaaS service (formerly Real-Time Analytics) |

**Setup in short:** create a watcher (subscription, resource group, name, region) → choose the data store → add SQL targets → create → press **Start**. A new watcher does **not** start by itself.

**Limits and costs (preview):**
- Up to **100 SQL targets per watcher** and **20 watchers per subscription**.
- The watcher and dashboards are **free**. You pay for the ADX cluster or Fabric capacity (and alerts, and Key Vault if you use SQL authentication).
- **Alerts** (Azure Monitor log search alerts) are **not** available if the data store is Fabric Real-Time Intelligence or a free ADX cluster.

**Example:** A software company has 60 customer databases across several elastic pools. It creates one database watcher with a paid ADX cluster. Now one dashboard shows CPU, waits and blocking for all 60 databases, and an alert fires when a deadlock happens.

## Intelligent Insights
**Intelligent Insights** uses built-in AI to watch your database and find events that hurt performance.
- Works for **Azure SQL Database** (single and pooled) and **Azure SQL Managed Instance**.
- Status: still **preview**, and not available in West Europe, North Europe, West US 1 and East US 1.
- When it finds a problem, it writes a log entry in the **`SQLInsights`** category with:
  - a **root cause analysis** (a plain-language reason),
  - the **affected queries** (as query hashes) or error codes,
  - **recommendations** where possible.

**How it works:**
- Compares the **last hour** of workload with a **7-day baseline**.
- Also checks fixed limits: too many waits, critical errors, parameterization problems.
- Analysis usually takes about **1 hour**. For a problem happening right now, use **Query Store** or DMVs.
- Each issue has a status: **Active → Verifying → Complete**.

**What it measures:** query duration, timed-out requests, excessive wait times, errored-out requests.

**Some of the 15 patterns it can detect** (others include workload increase, memory pressure, tempdb contention and slow client):
| Pattern | Typical fix |
|---------|-------------|
| Reaching resource limits | Tune queries or scale up |
| Locking | Shorter transactions, smaller batches |
| Increased MAXDOP | Set MAXDOP lower |
| Missing index | Create the index or turn on automatic tuning |
| Elastic pool DTU shortage | Tune or raise the pool size (**SQL Database only**, never on MI) |
| Plan regression | Force the good plan / **FORCE_LAST_GOOD_PLAN** |
| Database-scoped configuration change | Undo the recent setting change |
| Pricing tier downgrade | Scale back up or reduce load |

### Turning it on: diagnostic settings
Insights are **not stored by default**. You must add a **diagnostic setting** on **each database** and tick the **`SQLInsights`** category.

| Destination | Use for |
|-------------|---------|
| **Log Analytics workspace** | KQL queries, log alerts, workbooks (table `AzureDiagnostics`) |
| **Event Hubs** | Streaming to a SIEM or third-party tool |
| **Azure Storage** | Cheap long-term archive |

- **Managed Instance:** set it on each **instance database**. You cannot set it on MI **system databases**.
- **Elastic pool:** set it on **each pooled database**.

**Example:** A bank's payments database gets slower one afternoon. An hour later, Intelligent Insights logs: "Plan regression on query hash X". The bank already sends `SQLInsights` to Log Analytics, and a log alert emails the DBA. Automatic tuning had already forced the last good plan.

Note: Do not mix up the `SQLInsights` **log category** (still exists) with **Azure Monitor SQL Insights** (a separate product, retired 31 Dec 2024).

## Compare at a glance
| Need | Tool |
|------|------|
| Catch exact events such as deadlocks or one slow statement | Extended Events |
| Near real-time dashboards across many Azure SQL databases | Database watcher |
| AI root cause of a regression, streamed to Log Analytics or Event Hubs | Intelligent Insights |

## Remember for the exam
- Extended Events replaces SQL Profiler and has low overhead.
- Azure SQL Database XE sessions use **ON DATABASE**; SQL Server and MI use **ON SERVER**.
- In Azure SQL DB / MI the **event_file** target writes to **Blob Storage**; **ETW** and **pair_matching** are SQL Server only.
- **ring_buffer** = memory, lost when stopped. **event_file** = kept.
- Database watcher: SQL DB and MI only (not VMs), data in **ADX** or **Fabric RTI**, must press **Start**.
- No alerts with a **free ADX cluster** or **Fabric RTI** data store.
- Intelligent Insights: **7-day baseline**, about **1 hour** to analyse, output = **`SQLInsights`** log.
- Must enable a **diagnostic setting** per database. Archive → Storage, SIEM → Event Hubs, KQL and alerts → Log Analytics.
- **Elastic pool DTU shortage** never shows on Managed Instance.

## Quick check
1. You must capture deadlock details in Azure SQL Database and keep them after the session stops. Which target?
   <details><summary>Answer</summary>event_file (written to Azure Blob Storage). ring_buffer is in memory and is lost when the session stops.</details>
2. You need root cause analysis of performance regressions sent to a third-party SIEM. What do you configure?
   <details><summary>Answer</summary>A diagnostic setting on each database with the SQLInsights category sent to Event Hubs.</details>
3. Can database watcher monitor SQL Server running on an Azure VM?
   <details><summary>Answer</summary>No. It supports only Azure SQL Database and Azure SQL Managed Instance.</details>
