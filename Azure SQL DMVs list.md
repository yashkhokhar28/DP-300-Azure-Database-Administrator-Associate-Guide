## Azure SQL Dynamic Management Views (DMVs)

---

### **1. Plan and Implement Data Platform Resources**

| DMV                                  | Purpose                                                               | Azure SQL Notes                           | Real-World Example                      | Practice Question                                                     |
| ------------------------------------ | --------------------------------------------------------------------- | ----------------------------------------- | --------------------------------------- | --------------------------------------------------------------------- |
| `sys.dm_db_resource_stats`           | Shows CPU, data IO, log IO usage over the last hour (5-min intervals) | Available in Azure SQL Database only      | See which hour your DTU usage peaks     | **Q:** How would you identify peak CPU usage in the last hour?        |
| `sys.resource_stats`                 | CPU, IO, storage for the last 14 days (15-min intervals)              | Azure-only view, queryable from master DB | Track workload trends over 2 weeks      | **Q:** Which DMV lets you check IO usage trends for the past 2 weeks? |
| `sys.dm_user_db_resource_governance` | Resource limits for the DB (eDTUs, IO caps)                           | Azure-only                                | Check if your DB is hitting eDTU limits | **Q:** Which DMV shows current DTU limits?                            |

---

### **2. Implement a Secure Environment**

| DMV                       | Purpose                                          | Azure SQL Notes    | Real-World Example                     | Practice Question                                                       |
| ------------------------- | ------------------------------------------------ | ------------------ | -------------------------------------- | ----------------------------------------------------------------------- |
| `sys.dm_exec_connections` | Shows current connections, protocols, encryption | Same as SQL Server | Check if clients are using TLS 1.2     | **Q:** How do you verify TLS encryption is enabled for all connections? |
| `sys.dm_exec_sessions`    | Active sessions with login, app, host info       | Same               | Find suspicious logins from unknown IP | **Q:** Which DMV lists all active logins and their originating apps?    |
| `sys.dm_exec_requests`    | Running queries & their state                    | Same               | Identify a blocking query              | **Q:** Which DMV do you use to find the exact SQL causing blocking?     |

---

### **3. Monitor, Configure, and Optimize Database Resources**

| DMV                                 | Purpose                              | Azure SQL Notes | Real-World Example                       | Practice Question                                                         |
| ----------------------------------- | ------------------------------------ | --------------- | ---------------------------------------- | ------------------------------------------------------------------------- |
| `sys.dm_exec_query_stats`           | Aggregated query execution stats     | Same            | Find top 5 queries by CPU time           | **Q:** How do you find the most CPU-heavy query in the last hour?         |
| `sys.dm_exec_query_plan`            | XML execution plan for a query       | Same            | Check for missing index hints            | **Q:** Which DMV do you join with `sys.dm_exec_sql_text` to get plans?    |
| `sys.dm_db_index_usage_stats`       | Index seek/scan usage counts         | Same            | Drop unused indexes to save DTUs         | **Q:** How do you find indexes that haven’t been used since last restart? |
| `sys.dm_db_index_operational_stats` | Low-level index ops like page splits | Same            | Detect high fragmentation on hot indexes | **Q:** Which DMV shows page split counts?                                 |

---

### **4. Configure and Manage Automation of Tasks**

| DMV                                      | Purpose                                       | Azure SQL Notes | Real-World Example          | Practice Question                                             |
| ---------------------------------------- | --------------------------------------------- | --------------- | --------------------------- | ------------------------------------------------------------- |
| `sys.dm_exec_background_job_queue`       | Shows queued background jobs (Azure internal) | Azure-only      | Debug Elastic Job execution | **Q:** Which DMV do you use for queued Elastic Job debugging? |
| `sys.dm_exec_background_job_queue_stats` | Stats about background job execution          | Azure-only      | Identify job bottlenecks    | **Q:** Which DMV helps measure job run success vs failures?   |

---

### **5. Plan and Configure HA/DR Environment**

| DMV                                       | Purpose                          | Azure SQL Notes                       | Real-World Example                    | Practice Question                                               |
| ----------------------------------------- | -------------------------------- | ------------------------------------- | ------------------------------------- | --------------------------------------------------------------- |
| `sys.dm_hadr_database_replica_states`     | Sync state of DB replicas        | Same in Managed Instance & SQL Server | See if a secondary replica is lagging | **Q:** How do you find if a replica is in SYNCHRONIZED state?   |
| `sys.dm_hadr_availability_replica_states` | Health of each AG replica        | Same                                  | Check failover readiness              | **Q:** Which DMV shows replica failover mode?                   |
| `sys.dm_geo_replication_link_status`      | Status of active geo-replication | Azure-only                            | Find replication lag in seconds       | **Q:** Which DMV shows geo-replication latency in Azure SQL DB? |
