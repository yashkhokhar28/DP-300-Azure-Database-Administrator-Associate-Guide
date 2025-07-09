## 🔍 **What Are Dynamic Management Views (DMVs) and Functions (DMFs)?**

- **DMVs and DMFs** are built-in objects in SQL Server that let you **check what's happening inside the database and the server**.
- They help you **monitor, troubleshoot, and tune** SQL Server or Azure SQL Database by showing real-time internal data.

They **show things like:**

- How queries are running
- Current server health
- Active transactions
- Cached execution plans
- Resource usage (CPU, memory, I/O)

---

## 🛡️ **Permissions Required**

There are **two levels of DMVs**, depending on what they monitor:

| Scope               | What They Monitor                                                | Required Permission   |
| ------------------- | ---------------------------------------------------------------- | --------------------- |
| **Server Scoped**   | Entire SQL Server instance (CPU usage, overall wait times, etc.) | `VIEW SERVER STATE`   |
| **Database Scoped** | Specific database info (query stats, transactions, indexes)      | `VIEW DATABASE STATE` |

---

## ⚙️ **Naming Convention**

All DMVs/DMFs start with:

```
sys.dm_
```

and then include their **category and purpose**.

Example DMV names:

- `sys.dm_exec_requests` → Shows running queries.
- `sys.dm_exec_sessions` → Shows active sessions.
- `sys.dm_db_index_usage_stats` → Shows index usage.
- `sys.dm_exec_cached_plans` → Shows cached query plans.

---

## 📂 **Three Main Categories of DMVs**

| Category                    | Examples                                                   | Purpose                                                  |
| --------------------------- | ---------------------------------------------------------- | -------------------------------------------------------- |
| **Database-related**        | `sys.dm_db_index_usage_stats`, `sys.dm_db_partition_stats` | Show database-level info like index usage, object sizes. |
| **Query execution-related** | `sys.dm_exec_requests`, `sys.dm_exec_sessions`             | Show info about running queries and sessions.            |
| **Transaction-related**     | `sys.dm_tran_locks`, `sys.dm_tran_active_transactions`     | Show info about current transactions and locks.          |

---

## 🔄 **Differences Between Azure SQL Database and SQL Server DMVs**

- Some DMVs exist only in **SQL Server on-premises**, and some only in **Azure SQL Database**.
- Example differences:

  - **SQL Server only:** `sys.dm_os_wait_stats` (shows server-level wait statistics, not available in Azure SQL Database).
  - **Azure SQL only:** `sys.dm_db_resource_stats` (shows resource usage specific to Azure's environment).

---

## 🛠️ **Older Versions Without Query Store**

If you use older SQL Server versions (before Query Store existed), you can still see query plans using:

```sql
sys.dm_exec_cached_plans
sys.dm_exec_sql_text
sys.dm_exec_query_plan
```

But these only show **the current plan in the cache**, not historical plan changes over time (which Query Store provides).

---

## ✅ **Summary**

- **DMVs/DMFs** are powerful tools to monitor and troubleshoot SQL Server or Azure SQL.
- You’ll use them to:

  - Check query performance
  - Diagnose problems
  - Understand system/resource usage

- Different permission levels control who can see what.
- Azure SQL and SQL Server have some differences in what DMVs are available.

---
