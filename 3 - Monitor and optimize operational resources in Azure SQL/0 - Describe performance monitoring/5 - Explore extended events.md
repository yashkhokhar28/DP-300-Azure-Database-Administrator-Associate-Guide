## 🎯 **What Are Extended Events?**

Extended Events (**XEs**) are a **flexible event-handling system** in SQL Server and Azure SQL Database. They:

- Help you **troubleshoot performance problems** by capturing detailed activity.
- Can monitor nearly everything happening inside SQL Server (queries, locks, blocking, I/O issues, etc.).
- Allow you to filter and capture only what you need, reducing performance overhead.

---

## 🔍 **What Can You Monitor with Extended Events?**

| **Example Use Cases**                                       |
| ----------------------------------------------------------- |
| Troubleshooting **deadlocks** or blocking                   |
| Finding **long-running queries**                            |
| Watching for **DDL operations** (e.g., table modifications) |
| Logging **missing statistics**                              |
| Tracking **memory pressure** or **slow disk I/O**           |

---

## 🛠️ **Key Parts of an Extended Events Session**

| **Component** | **Purpose**                                                                      |
| ------------- | -------------------------------------------------------------------------------- |
| **Event**     | A specific thing to monitor (e.g., a query completes, a lock is acquired).       |
| **Action**    | Extra data captured when the event occurs (e.g., session ID, query text).        |
| **Filter**    | Limits which events get collected (e.g., only user queries, not system queries). |
| **Target**    | Where to store the data (e.g., in memory, a file, Azure blob storage).           |

---

## 📊 **Event Categories**

Extended Events are organized into **four channels** based on their purpose:

| Channel         | Purpose                      | Example                        |
| --------------- | ---------------------------- | ------------------------------ |
| **Admin**       | Common troubleshooting       | Deadlock report                |
| **Operational** | Diagnostics, state changes   | Database failover              |
| **Analytic**    | High-volume performance data | Query execution trace          |
| **Debug**       | Low-level debug data         | For Microsoft support use only |

---

## ⚙️ **How to Create an Extended Events Session (Graphically in SSMS)**

1. **In SSMS**: Go to **Management → Extended Events → Sessions → New Session.**
2. Name the session.
3. Choose from templates like:

   - **Locks and Blocks**
   - **Query Execution**
   - **Profiler Equivalents**
   - **System Monitoring**

4. Select which **events** to track (e.g., `sql_statement_completed`, `sql_batch_completed`).
5. Add **actions** (e.g., collect session ID, SQL text).
6. Add **filters** to narrow down the events (e.g., only user sessions, exclude system activity).
7. Choose a **target**, such as:

   - **Event File** → saves events to a file for later review.
   - **Ring Buffer** → keeps data temporarily in memory for real-time troubleshooting.

---

## ⚡ **Storage Targets Explained**

| **Target Type**                     | **Purpose**                                                            | **Processing Type** |
| ----------------------------------- | ---------------------------------------------------------------------- | ------------------- |
| **Event File**                      | Persistently saves event data to disk (or Azure Blob Storage in PaaS)  | Asynchronous        |
| **Ring Buffer**                     | Stores events in memory temporarily (lost when session ends)           | Asynchronous        |
| **Event Counter**                   | Counts occurrences of each event                                       | Synchronous         |
| **ETW (Event Tracing for Windows)** | Correlates SQL events with OS-level traces                             | Synchronous         |
| **Histogram**                       | Shows counts by specific field values (like response times)            | Asynchronous        |
| **Event Pairing**                   | Identifies when paired events (e.g., lock acquire/release) don’t match | Asynchronous        |

---

## 🔑 **T-SQL Example to Create a Session**

```sql
IF EXISTS (SELECT * FROM sys.server_event_sessions WHERE name='test_session')
    DROP EVENT SESSION test_session ON SERVER;
GO

CREATE EVENT SESSION test_session
ON SERVER
    ADD EVENT sqlos.async_io_requested,
    ADD EVENT sqlserver.lock_acquired
    ADD TARGET package0.etw_classic_sync_target
        (SET default_etw_session_logfile_path = N'C:\demo\traces\sqletw.etl')
    WITH (MAX_MEMORY=4MB, MAX_EVENT_SIZE=4MB);
GO
```

- This session watches for async I/O and lock acquisition.
- It outputs data to an ETW trace file.

---

## ✔️ **Best Practices for Using Extended Events:**

- Always **use filters** to reduce overhead.
- Store data to an **event file** for long-running sessions.
- Use **ring buffer** for quick, real-time troubleshooting.
- Only track events you really need.
- Start/stop sessions as needed through SSMS or T-SQL.

---

## ✅ **Key Takeaways:**

- Extended Events are the **modern, efficient replacement for SQL Profiler**.
- You can **troubleshoot nearly anything in SQL Server** using XEs.
- XEs allow for **minimal overhead, flexible filtering**, and can store data where you need it.
- Perfect for **diagnosing tough performance issues** like deadlocks, blocking, or slow queries.

---
