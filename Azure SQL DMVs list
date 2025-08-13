
# Azure SQL Session-Related Dynamic Management Views (DMVs)

This document lists **all session-related DMVs** available in **Azure SQL Database** and **Azure SQL Managed Instance**, including their purpose and real-world use cases.  
These DMVs help track connection state, activity, resource usage, and query details at the session level.

---

## Table of Session-Related DMVs

| DMV Name | Description | Real-World Example |
|----------|-------------|--------------------|
| **sys.dm_exec_sessions** | Lists all active sessions (user and internal) with login, host, program, login time, and TTL. | Identify idle but connected sessions consuming resources. |
| **sys.dm_exec_connections** | Shows physical connection details (protocol, IP, encryption, connect time). | Detect insecure connections or connections from unexpected IP ranges. |
| **sys.dm_exec_requests** | Displays current executing requests per session with status, timings, and wait info. | Find and terminate a blocking query that's slowing down the app. |
| **sys.dm_exec_requests_history** *(Azure-only)* | Provides historical data of requests per session. | Audit activity for a noisy tenant in a multi-tenant database. |
| **sys.dm_db_session_space_usage** | Tracks TempDB space (pages allocated/deallocated) per session. | Find which session is consuming excessive TempDB space. |
| **sys.dm_db_task_space_usage** | Shows TempDB usage per task within sessions. | Isolate a subtask inside a session that's bloating TempDB. |
| **sys.dm_exec_cursors (session_id)** | Lists open cursors for a given session. | Identify abandoned cursors slowing down workloads. |
| **sys.dm_exec_input_buffer (session_id, request_id)** | Reveals the last T-SQL statement executed by a session. | Catch ad-hoc, unoptimized queries causing performance hits. |
| **sys.dm_exec_query_plan (plan_handle)** | Returns the execution plan for a request in a session. | Diagnose slow queries due to parameter sniffing or missing indexes. |
| **sys.dm_exec_query_stats** | Provides aggregated query performance stats including CPU, I/O, execution counts. | Determine which session’s queries are the biggest CPU consumers. |

---

## Typical Workflow for Session Monitoring

1. **Check who’s connected**  
   Use `sys.dm_exec_sessions` + `sys.dm_exec_connections` to see login details and network origin.

2. **Find active queries**  
   Query `sys.dm_exec_requests` to see what’s running right now.

3. **Look at history (Azure-only)**  
   Use `sys.dm_exec_requests_history` to find past activity even if the session is gone.

4. **Inspect TempDB usage**  
   Check `sys.dm_db_session_space_usage` and `sys.dm_db_task_space_usage` for excessive allocations.

5. **See exactly what was run**  
   Pull the last query from `sys.dm_exec_input_buffer`.

6. **Analyze performance**  
   Use `sys.dm_exec_query_stats` + `sys.dm_exec_query_plan` to find tuning opportunities.

---

## Example Query: Find Top CPU-Consuming Sessions

```sql
SELECT 
    s.session_id,
    s.login_name,
    c.client_net_address,
    r.status,
    r.cpu_time,
    r.total_elapsed_time,
    t.text AS query_text
FROM sys.dm_exec_sessions s
JOIN sys.dm_exec_connections c 
    ON s.session_id = c.session_id
JOIN sys.dm_exec_requests r 
    ON s.session_id = r.session_id
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) t
ORDER BY r.cpu_time DESC;
````

**Use Case:**
Quickly identify which session is consuming the most CPU and what query it’s running.

---

## Permissions Required

* **Azure SQL Database:** `VIEW DATABASE STATE`
* **Azure SQL Managed Instance:** `VIEW SERVER STATE`

---

*Reference: [Microsoft Learn – Dynamic Management Views in Azure SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-with-dmvs)*

```
