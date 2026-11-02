# DP-300 Cheat Sheet

> Read this the night before the exam. ~30 min.

Short names used below: **VM** = SQL Server on an Azure virtual machine (IaaS, you run it). **MI** = Azure SQL Managed Instance (PaaS). **SQL DB** = Azure SQL Database (PaaS). **PaaS** = Microsoft runs the server for you. **IaaS** = you get a machine and run SQL Server yourself.

## 1. Which Azure SQL option?

| | SQL Server on VM | SQL Managed Instance | Azure SQL Database | SQL database in Fabric | SQL Server enabled by Azure Arc |
|---|---|---|---|---|---|
| Type | IaaS | PaaS | PaaS | SaaS, inside Microsoft Fabric | Your own server (on-prem, other cloud, edge) managed from Azure |
| OS access | Yes | No | No | No | Yes (it is your server) |
| Code changes to move | None | Few or none | Many | n/a | None (it does not move) |
| SQL Agent | Yes | Yes (always on) | No (use elastic jobs) | No | Yes |
| Cross-database queries, linked servers, CLR | Yes | Yes | Limited | No | Yes |
| Admin effort | High | Lower | Lowest | Lowest | Same as today |
| Best for | Lift-and-shift as-is, old SQL versions, OS-level needs, SSRS/SSAS, FILESTREAM, PolyBase | Moving an existing app with few changes | New cloud apps, SaaS, unknown or spiky load | Analytics on your SQL data without ETL | Keeping servers where they are but managing them from Azure |

**If the question says X → pick Y**

- Needs OS access, a custom SQL version or patch level → **VM**.
- Needs SSRS, SSAS, FILESTREAM, PolyBase or cross-instance transactions → **VM**.
- Needs SQL Agent, linked servers, cross-database queries, CLR, with **no OS work** → **MI**.
- "Minimal code changes" + "least admin effort" → **MI**.
- New app, auto-scale, pay only when used → **SQL DB serverless** (General Purpose or Hyperscale).
- Database bigger than 4 TB, up to 128 TB → **SQL DB Hyperscale**. Hyperscale is now Microsoft's recommended default tier for new OLTP work.
- Many small databases with uneven load (for example, one per customer) → **SQL DB elastic pool**.
- Data must be copied to **OneLake** for reporting and machine learning with no ETL (no copy jobs) → **SQL database in Fabric**. It creates a SQL analytics endpoint and a default semantic model.
- On-premises servers need inventory, best-practice checks, Defender alerts or Purview governance, **without migrating** → **Azure Arc**.
- IoT or edge device needs a small local database → **Azure SQL Edge**.

**Licensing quick picks**

| Need | Pick |
|---|---|
| Short test, no licence | Pay-as-you-go (licence included) |
| Already own SQL licences with Software Assurance | Azure Hybrid Benefit |
| VM running 1–3 years | Reserved capacity |
| Free licence for a passive DR copy | HA/DR licence type (VM) or a **standby** replica (SQL DB geo-secondary, MI failover group) |
| Hyperscale | No SQL licence fee. Azure Hybrid Benefit is **not** available for new Hyperscale databases |

**Size and tier numbers to know**

| Fact | Value |
|---|---|
| SQL DB General Purpose / Business Critical max size | 4 TB |
| SQL DB Hyperscale max size | 128 TB |
| Hyperscale back to General Purpose | Only within **45 days** |
| Hyperscale replicas | Up to 4 HA replicas, up to 30 named replicas, plus geo-replicas |
| Serverless auto-pause delay (General Purpose) | 15 min to 7 days, or -1 to turn off |
| Things that stop serverless auto-pause | Geo-replication, failover groups, long-term retention, elastic job database, Data Sync |
| MI public endpoint port | 3342 |
| MI stop/start | General Purpose only. You still pay for storage |

**VM storage quick picks**: Data files on Premium SSD, Premium SSD v2 or Ultra Disk. Read caching **on** for data disks, **off** for log disks. tempdb on the local **D:** drive (it is wiped on restart). Standard HDD is for backups only. Production sizes: memory-optimized, for example **Ebdsv5**. Constrained-core VMs keep the RAM and disk speed but cut SQL licence cost.

## 2. IaaS vs PaaS: who does what

**Security**

| Area | VM (IaaS) | SQL DB / MI (PaaS) |
|---|---|---|
| Login types | SQL auth, Windows auth (Active Directory) | SQL auth, **Microsoft Entra ID** (with MFA). Windows auth for Entra users only on **MI** |
| TDE (encrypts the files on disk) | You turn it on | **On by default** |
| Network | NSGs, Windows Firewall, VPN / ExpressRoute | Server and database firewall rules (SQL DB), service endpoints, Private Link. MI uses NSGs |
| Patching | You | Microsoft |
| Threat detection | Defender for SQL Servers on Machines | Defender for Azure SQL Databases |

**Automation and deployment**

| Area | VM (IaaS) | SQL DB / MI (PaaS) |
|---|---|---|
| Install SQL | You (or a Marketplace image) | Azure does it |
| OS and SQL patches | You. Use **Azure Update Manager** | Automatic |
| Job scheduler | SQL Agent | MI: SQL Agent. SQL DB: **elastic jobs** |
| Deploy as code | ARM, Bicep, Terraform, PowerShell, CLI | Same, plus DACPAC in Azure DevOps / GitHub Actions |

**Performance**

| Area | VM (IaaS) | SQL DB / MI (PaaS) |
|---|---|---|
| Query Store | Turn on yourself (SQL Server 2016–2019). On by default for new DBs in SQL Server 2022+ | On by default. Cannot be turned off in SQL DB |
| MAXDOP | `sp_configure` (instance) | SQL DB: `ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP` |
| Scale up | Resize the VM (VM restarts) | Change vCores or tier in seconds to minutes |
| OS counters (PerfMon) | Yes | No |
| Auto tuning | No | Yes (plan correction, index tuning on SQL DB) |
| Index and statistics maintenance | You | Still you (backups and DBCC checks are done by the platform) |

**Backups**

| Area | VM (IaaS) | SQL DB | MI |
|---|---|---|---|
| Who backs up | You (or Azure Backup / IaaS Agent) | Azure, automatic | Azure, automatic |
| Your own backups | Yes: full, differential, log | No | Yes, but only `COPY_ONLY` to URL |
| Restore point in time | Yes, if FULL recovery model and log backups | 1–35 days (default 7) | 1–35 days (default 7) |
| Long-term retention | Azure Backup or your own storage | Up to 10 years | Up to 10 years |

**HA/DR**

| Area | VM (IaaS) | SQL DB / MI (PaaS) |
|---|---|---|
| HA in one region | Always On AG, FCI (you build it) | Built in, nothing to set up. Zone redundancy is an option |
| DR to another region | AG async replica, distributed AG, log shipping, ASR, backups in GRS storage | SQL DB: active geo-replication or failover groups. MI: failover groups (or MI link) |
| Connection string after DR failover | You fix DNS or the string | Same name with failover group listener |

## 3. Migration tool chooser

**Step 1: assess.** Use **Azure Migrate** or the **SQL Server enabled by Azure Arc** assessment.

| Source → target | Online / offline | Tool | Key facts |
|---|---|---|---|
| SQL Server → VM | Offline | **Azure Migrate** (move the whole server) | Lift-and-shift |
| SQL Server → VM | Offline | **Backup to URL + restore** | `BACKUP DATABASE ... TO URL` |
| SQL Server → VM | Online | **Log shipping** or **Always On AG** | Add the Azure replica with **T-SQL** (the Add Azure Replica Wizard was for classic VMs, retired) |
| SQL Server → VM | Offline | **Detach/attach**, **Azure Import/Export service** (ship disks) | Very large or slow-network moves |
| SQL Server → MI | Online | **Managed Instance link** | SQL Server 2016+. Cutover in seconds. Target readable. No time limit. Two-way failback with SQL Server 2022/2025 |
| SQL Server → MI | Online | **Log Replay Service (LRS)** | SQL Server 2012+, all editions. Must finish in **30 days**. Target not readable. Modes: Autocomplete or Continuous |
| SQL Server → MI | Online or offline | **Azure DMS** (portal, CLI `az datamigration`, PowerShell) | Large, complex moves |
| SQL Server → MI | Offline | **Native backup/restore** `RESTORE ... FROM URL` | Simple |
| SQL Server (Arc) → MI | Online | **Arc migration in the portal** | Uses MI link or LRS inside |
| SQL Server → SQL DB | Online | **Transactional replication** | The **only online** way into SQL DB. Tables need a primary key. SQL DB is a **push subscriber** only |
| SQL Server → SQL DB | Offline | **Azure DMS** | Large databases |
| SQL Server → SQL DB / MI | Offline | **BACPAC + SqlPackage** | Schema + data. Over ~150 GB, use SqlPackage, not the portal. MI cannot import a BACPAC from the portal |
| Some tables only | Offline | **bcp**, **Azure Data Factory** | Partial or scheduled copies |
| SQL DB → MI | Offline | **Azure DMS** | Between Azure SQL services |

**Retired tools** (the exam may still name them):

| Tool | Retired | Use instead |
|---|---|---|
| Data Migration Assistant (DMA) | July 2025 | Azure Migrate or Arc assessment, DMS |
| Azure Data Studio and its migration extension | 28 Feb 2026 | Azure DMS (portal/CLI/PowerShell), SSMS |
| Database Experimentation Assistant (DEA) | 15 Dec 2024 | n/a |

**Migration tips**: Choose online when downtime must be near zero (hospital, 24/7 shop). Scale the target up during the move, then scale down. After migration, check for plan regressions (new cardinality estimator), missing indexes and parameter sniffing.

## 4. Security quick picks

| Need | Pick |
|---|---|
| One identity, MFA, single sign-on | **Microsoft Entra ID** authentication |
| Turn off SQL logins completely | **Microsoft Entra-only authentication** (set an Entra admin first) |
| App connects with no password | **Managed identity** |
| User that moves with the database | **Contained user** (`CREATE USER ... FROM EXTERNAL PROVIDER`) |
| Server-level roles in SQL DB | Fixed `##MS_...##` roles (for example `##MS_DatabaseManager##`, `##MS_LoginManager##`), set with T-SQL in `master` |
| Create databases but not read data | **SQL DB Contributor** (Azure RBAC) |
| Manage security settings but not read data | **SQL Security Manager** |
| Block access even if granted elsewhere | `DENY` (beats `GRANT`). `REVOKE` just removes a grant or deny |
| Encrypt whole database and backups at rest | **TDE** (AES-256). Key in Key Vault = **customer-managed key (BYOK)** |
| Hide column data even from DBAs | **Always Encrypted** (keys stay with the client app) |
| Always Encrypted, need joins/equality | **Deterministic** encryption |
| Always Encrypted, most secure | **Randomized** (rich queries need **secure enclaves**) |
| Encrypt data on the network | **TLS** (`Encrypt=True;TrustServerCertificate=False`). Minimum TLS is now **1.2** |
| Show `a****@g****.com` to call-centre staff | **Dynamic Data Masking**. Grant `UNMASK` to people who need real values |
| Each tenant sees only its own rows | **Row-Level Security** (security policy + predicate function) |
| Label columns as Confidential, GDPR, etc. | **Data Discovery & Classification** (`ADD SENSITIVITY CLASSIFICATION`) |
| Org-wide map of sensitive data and lineage | **Microsoft Purview** |
| Tamper-proof history for auditors | **Ledger** tables (digests in immutable Blob or Azure Confidential Ledger) |
| Log who did what | **Auditing** to Storage (cheap), Log Analytics (query), Event Hubs (real-time) |
| Alerts on SQL injection, brute force, odd logins + vulnerability scan | **Microsoft Defender for SQL** |
| Keep row history over time | **Temporal tables** |
| Know which rows changed (no old values) | **Change tracking** |
| Keep full change history for ETL | **Change data capture (CDC)** |
| Store keys and secrets | **Azure Key Vault** (role **Key Vault Crypto Service Encryption User**) |
| Stop SQL injection | Parameterized queries or stored procedures (keyword filters are not enough) |

**Network quick picks**

| Need | Pick |
|---|---|
| Whole company reaches all DBs on a server | **Server-level firewall rule** (`sp_set_firewall_rule` in `master`) |
| One partner reaches one database | **Database-level firewall rule** (`sp_set_database_firewall_rule`) |
| Only one VNet subnet may connect, public endpoint still exists | **Service endpoint** |
| No public access at all, private IP inside the VNet | **Private Link** (private endpoint) |
| Encrypted tunnel over the internet, low cost | **VPN Gateway** |
| Private fibre, high speed, compliance | **ExpressRoute** |
| Lowest latency inside Azure | Connection policy **Redirect** (ports 11000–11999). **Proxy** uses 1433 only |

Firewall check order: database rule first, then server rule, then block. Limits: 256 server-level and 256 database-level IP rules. "Allow Azure services" makes a `0.0.0.0` rule that lets in **every** Azure customer's services.

**TDE build order (VM)**: Service Master Key → `CREATE MASTER KEY` → `CREATE CERTIFICATE` → `CREATE DATABASE ENCRYPTION KEY` → `ALTER DATABASE ... SET ENCRYPTION ON`. For an AG, copy the certificate to every replica.

## 5. Performance: Query Store, plans, waits, DMVs

**Query Store** (a recorder of queries, plans and run stats; survives restarts, unlike DMVs).

| Setting | Default | Remember |
|---|---|---|
| Operation mode | READ_WRITE | Goes **READ_ONLY** when Max Size is full |
| Data flush interval | 15 min (900 s) | How often memory is written to disk. The interval tied to **write pressure** |
| Statistics collection interval | 60 min | Only the size of time buckets. Not linked to read-only |
| Max size | 1,000 MB | Raise it if Query Store keeps going read-only |
| Stale query threshold | 30 days | Old data is cleaned up |
| Capture mode | AUTO | Skips tiny one-off queries |

- Query Store read-only → raise **Max Size**, keep size-based cleanup on, or lower the stale threshold.
- "Which interval affects write pressure?" → **Data flush interval**, not statistics interval.
- Plan got worse after a change → Query Store **Regressed Queries**, then **force** the old plan (`sp_query_store_force_plan`).
- Fix a vendor app's query without changing its code → **Query Store hints** (`sp_query_store_set_hints`).
- Automatic tuning needs Query Store **ON** and **READ_WRITE**:

```sql
ALTER DATABASE [Db1] SET QUERY_STORE = ON;
ALTER DATABASE [Db1] SET QUERY_STORE (OPERATION_MODE = READ_WRITE);
ALTER DATABASE [Db1] SET AUTOMATIC_TUNING = AUTO;
```

**Automatic tuning defaults**: FORCE_LAST_GOOD_PLAN **on**. CREATE_INDEX and DROP_INDEX **off**. MI supports **only** FORCE_LAST_GOOD_PLAN. Suggestions are in `sys.dm_db_tuning_recommendations`.

**Estimated vs actual plan**

| | Estimated plan | Actual plan |
|---|---|---|
| When | Before the query runs | After it runs |
| Shows | Estimated rows and costs | Estimated **and** actual rows, spills, warnings |
| Risk | None | The query really runs |
| SSMS key | Ctrl+L | Ctrl+M |

Estimated rows far from actual rows → **stale statistics** or a bad plan. Run `UPDATE STATISTICS`.

**Plan problems → fix**

| Problem | Fix |
|---|---|
| Table or clustered index **scan** for a small result | Add a nonclustered index on the filter column |
| **Key lookup** | Covering index with `INCLUDE` columns |
| Non-SARGable filter (`WHERE CAST(col ...)`) | Rewrite so the column is not wrapped in a function |
| Parameter sniffing (fast for one value, slow for another) | `OPTION (RECOMPILE)`, `OPTIMIZE FOR UNKNOWN`, or **PSP optimization** (compat 160) |
| Blocking from readers and writers | **RCSI** or snapshot isolation (RCSI is on by default in SQL DB) |
| Last-page insert contention (`PAGELATCH_EX` on IDENTITY key) | `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON` |
| Long index rebuild that must survive failover | **Resumable** index rebuild (`RESUME`, `ABORT`, `MAX_DURATION`) |
| Big analytics scans | **Columnstore** index |

**Intelligent Query Processing (IQP)** turns on by raising **compatibility level** (140/150/160/170). New SQL DB databases default to **170**. Features: adaptive joins, memory grant feedback, scalar UDF inlining (150), table variable deferred compilation, approximate count distinct, PSP optimization, DOP feedback and CE feedback (160).

**Wait types**

| Wait | Means |
|---|---|
| `PAGEIOLATCH_SH` | Slow disk reads |
| `PAGELATCH_UP` on tempdb | tempdb allocation contention (no disk I/O) → more tempdb files |
| `RESOURCE_SEMAPHORE` | Queries waiting for memory |
| `LCK_M_X` / `LCK_M_*` | Blocking (locks) |
| `CXPACKET` | Parallelism to tune (MAXDOP, cost threshold). `CXCONSUMER` is normal |
| `LOG_RATE_GOVERNOR` | SQL DB is capping log write speed → smaller transactions or bigger tier |
| `INSTANCE_LOG_RATE_GOVERNOR` | MI instance log limit |
| `THREADPOOL` | Not enough worker threads |
| `HADR_SYNC_COMMIT` | Business Critical waiting for replica sync |
| `HADR_THROTTLE_LOG_RATE_MISMATCHED_SLO` | Geo-secondary is smaller than the primary |

Error **10928** in SQL DB = worker limit reached → scale up.

**Top DMVs** (permission: `VIEW DATABASE STATE` in SQL DB, `VIEW SERVER STATE` on MI/VM)

| DMV | What it shows | Use it when |
|---|---|---|
| `sys.dm_db_resource_stats` | CPU, data I/O, log I/O every **15 s**, about **1 hour** kept (SQL DB) | "Is my database at its limit right now?" |
| `sys.resource_stats` (in `master`) | **5-minute** averages for every DB on the server, about 14 days | Capacity planning over days |
| `sys.elastic_pool_resource_stats` | Pool CPU, I/O, memory | Right-size an elastic pool |
| `sys.dm_user_db_resource_governance` | Limits of your service tier | Check what the SKU allows |
| `sys.dm_exec_requests` | Queries running now, their waits and blockers | Find blockers and long runners |
| `sys.dm_exec_sessions` | Who is connected, program, login time | Spot noisy apps or odd logins |
| `sys.dm_exec_connections` | Client IP, protocol, encrypted or not | Check connections are encrypted |
| `sys.dm_exec_input_buffer` | Last statement a session sent | What did that session run? |
| `sys.dm_exec_sql_text` | Full query text from a handle | Turn a handle into readable SQL |
| `sys.dm_exec_query_plan` | Cached plan XML | See scans, lookups, spills |
| `sys.dm_exec_query_stats` | CPU, reads, writes per cached plan | Top CPU or I/O queries since cache start |
| `sys.dm_exec_cached_plans` | Plan cache use and reuse | Too many single-use ad-hoc plans |
| `sys.dm_exec_query_plan_stats` | Last actual plan (opt-in: `LAST_QUERY_PLAN_STATS = ON`) | Actual plan without re-running |
| `sys.dm_db_wait_stats` | Waits for this database (SQL DB) | Top waits in SQL DB |
| `sys.dm_os_wait_stats` | Waits for the instance (MI, VM) | Top waits on MI or VM |
| `sys.dm_tran_locks` | Locks held and wanted | Blocking chains |
| `sys.dm_io_virtual_file_stats` | Read/write latency per file | Slow storage, slow AG redo |
| `sys.dm_db_index_usage_stats` | Seeks, scans, lookups, updates per index | Find unused indexes to drop |
| `sys.dm_db_index_physical_stats` | Fragmentation | Decide reorganize vs rebuild |
| `sys.dm_db_missing_index_details` / `_group_stats` | Suggested indexes and their impact | Which index to create first |
| `sys.dm_db_session_space_usage` | tempdb pages per session | Find the tempdb hog |
| `sys.dm_db_log_space_usage` | Log used vs total | Big transaction filling the log |
| `sys.dm_db_tuning_recommendations` | Automatic tuning actions and state | Did auto-tuning fix the regression? |
| `sys.dm_database_encryption_keys` | TDE state per database | Is TDE on after a restore? |
| `sys.dm_operation_status` (in `master`) | Long operations: restore, copy, scale | Track restore or copy progress |
| `sys.dm_geo_replication_link_status` | Geo-replication state and lag | Check geo-rep before failover |
| `sys.dm_database_backups` | Backup history (SQL DB) | Confirm backups for restore questions |
| `sys.dm_hadr_database_replica_states` | AG sync state, redo, lag | Replica lag before a planned failover |

**Quick troubleshooting flow**: App slow → check `sys.dm_db_resource_stats`. High CPU → Query Store top queries. Normal CPU → wait stats. Top queries in the portal (SQL DB only) → **Query Performance Insight**. Deep tracing (deadlocks) → **Extended Events** (in SQL DB use `ON DATABASE` sessions). Monitoring dashboard for many SQL DB/MI → **Database watcher** (data goes to Azure Data Explorer or Fabric Real-Time Intelligence; not for VMs).

**tempdb**: VM = one file per core up to 8, same size. SQL DB = set by Azure, cannot change. MI = 12 files by default, can change.

## 6. Automation quick picks

| Tool | Where it runs | Best for | Example |
|---|---|---|---|
| **SQL Server Agent** | VM, MI (not SQL DB) | Scheduled T-SQL jobs in one instance | Nightly index rebuild |
| **Elastic jobs** | SQL DB (all tiers) | Same T-SQL on many databases, across servers, pools, subscriptions | Add a column to 30 tenant DBs |
| **Azure Automation** runbooks | Azure, or on-prem via Hybrid Runbook Worker | PowerShell / Python scripts on a schedule | Start VM, run a procedure, stop VM |
| **Logic Apps** | Azure (serverless, low-code) | Event-driven flows that link many services | New order row → send email |
| **Azure Functions** | Azure (serverless code) | Small code triggered by events or alerts | Fix job called by an alert |
| **Azure Monitor alert + action group** | Azure | React to metrics or logs | CPU over 90% → email + runbook |
| **Maintenance Plan Wizard** (SSMS) | VM, MI | Backups, index and stats jobs without scripting | Stored as SSIS packages, jobs in `msdb` |
| **Multi-server (master/target) jobs** | VM | Push one job to many servers | 50 servers, one job |

**Elastic jobs facts**: Parts = job agent, **job database** (stores jobs), target group, job steps (T-SQL). Job database must be new and empty; not Hyperscale, not auto-pause serverless. Best auth = **user-assigned managed identity** (or database-scoped credentials). Agent tiers JA100 (default) to JA800. History kept **45 days**. Scripts should be **idempotent** (safe to run twice). Alert on failures with the **Failed Elastic jobs** metric.

**Agent facts**: On MI the Agent is always running and its settings are read-only. MI Database Mail profile must be named **`AzureManagedInstance_dbmail_profile`**. SQL Agent **alerts** are not supported on MI (only job email notifications) → use **Azure Monitor alerts**. Use a **proxy account** when a job step needs other rights.

**Deploy as code**

| Need | Pick |
|---|---|
| Repeatable deployment, desired state (declarative) | **Bicep** (Microsoft's recommended language) or ARM JSON |
| Quick one-off change, step by step (imperative) | PowerShell or Azure CLI |
| Turn ARM JSON into Bicep | `az bicep decompile` |
| Run on every code push | Azure DevOps Pipelines or GitHub Actions |
| Schema changes as a package | DACPAC (SqlPackage) |
| No local install | Azure Cloud Shell |

**Patching**: PaaS is automatic. VM → **Azure Update Manager** (SQL IaaS Agent Automated Patching retires 17 Sep 2027).

## 7. HA/DR quick picks

**RPO** = how much data you may lose (time). **RTO** = how long you may be down. If backups run hourly but RPO is 5 minutes, you fail. Use frequent log backups or replicas.

**Option chooser**

| Need | Pick | Data loss / failover |
|---|---|---|
| HA inside a region, PaaS | Built-in HA (nothing to do) | Automatic |
| Survive a zone failure, PaaS | **Zone redundancy** | Automatic |
| Readable copies in other regions, SQL DB, you fail over | **Active geo-replication** (up to **4** secondaries) | Async, manual, new connection string |
| Group of DBs fail over together, same connection string | **Failover group** (SQL DB or MI) | Async, listener names stay the same |
| DR for **MI** | **Failover group** (MI has no geo-replication) | One group per instance, all user DBs |
| HA for some databases on VMs + readable secondaries | **Always On AG** | Sync = no data loss in region |
| Protect the whole instance (logins, jobs) on VMs | **FCI** (needs shared storage) | Automatic, one copy of data |
| AGs in two regions, each with its own cluster | **Distributed AG** | Cross-region |
| Cheap, simple DR, manual | **Log shipping** | Some data loss, manual |
| Copy whole VMs (app + SQL) to another region | **Azure Site Recovery** | Not SQL-aware |
| On-prem SQL Server with DR in Azure | AG to an Azure VM, or **MI link** (two-way with SQL Server 2022/2025) | Over VPN or ExpressRoute |

**Geo-replication vs failover groups**

| | Active geo-replication | Failover groups |
|---|---|---|
| Works on | SQL DB only | SQL DB and MI |
| Scope | One database | Many DBs (SQL DB); all user DBs (MI) |
| Secondaries | Up to 4 readable | One (more is preview for SQL DB) |
| Connection after failover | Change the connection string | Same **read-write** and **read-only listener** names |
| Automatic failover | No | Only with the **Microsoft managed** policy (grace period min **1 hour**). **Customer managed** is recommended |
| Check health | `sys.dm_geo_replication_link_status` | Same, plus portal |

Rule: secondary must use the **same service tier**. Same compute size is strongly recommended.

**SLA numbers**: SQL DB 99.99%. Zone-redundant Business Critical/Premium 99.995%. VMs in availability zones 99.99%. VMs in availability sets 99.95%. Zone redundancy is not in Basic or Standard tiers.

**VM cluster facts**: Use **Cloud Witness** for quorum (disk witness when an FCI uses Azure shared disks). Put replicas in **multiple subnets** (no load balancer needed). In one subnet, use a **DNN** listener, or a VNN + **Standard** Load Balancer (Basic retired 30 Sep 2025). FCI shared storage = Azure shared disks, Premium file shares, Storage Spaces Direct, or Elastic SAN (2022+). AG = up to 9 replicas. **SQL Server 2022 contained AGs** carry logins and Agent jobs with the AG.

**Storage redundancy**

| Type | Copies | Survives | Read the copy? |
|---|---|---|---|
| **LRS** | 3 in one datacenter | Disk or rack failure | n/a. Cheapest. Not for DR |
| **ZRS** | 3 across zones in one region | Zone failure, not region | n/a |
| **GRS** | 6: 3 local + 3 in the paired region | Region failure | **No**, only after failover |
| **RA-GRS** | Same as GRS | Region failure | **Yes**, any time |
| **GZRS** | Zones in primary + paired region | Zone and region failure | No (RA-GZRS = yes) |

PaaS backup storage is **geo-redundant by default**. You can pick LRS, ZRS, GRS or GZRS. **Geo-restore** works only with GRS or GZRS. For Hyperscale, choose backup redundancy at creation only.

**PaaS backup numbers**

| Item | Value |
|---|---|
| Full backup | Weekly |
| Differential | Every 12 h (vCore) or 24 h (DTU) |
| Log backup | About every 5–10 min |
| Point-in-time restore window | **1–35 days**, default **7** (Basic max 7) |
| Long-term retention (LTR) | Weekly, monthly, yearly fulls, up to **10 years** |
| Hyperscale | Storage **snapshots**, not full/diff/log |
| Restore result | Always a **new** database (no restore in place) |
| Deleted logical server | Its PITR backups are gone. **LTR backups stay** |
| Deleted managed instance | Contact support within 5 days |
| SQL DB backup → SQL Server | Not possible |
| MI copy-only backup → SQL Server 2022/2025 | Possible when the MI update policy matches |
| SQL Server backup → MI | `RESTORE ... FROM URL` |
| MI `COPY_ONLY` with service-managed TDE key | Not allowed. Use a customer-managed key |

**VM backup options**

| Option | Facts |
|---|---|
| **Azure Backup for SQL in Azure VM** | Recovery Services vault, log backups every **15 min**, point-in-time restore of one DB, long-term retention |
| **Automated Backup** (SQL IaaS Agent extension) | Retention **1–90 days**, log every 5–60 min, needs FULL recovery model, SQL Server 2016+ |
| **Backup to URL** | `BACKUP DATABASE ... TO URL`. Use **SAS** + block blobs. One block blob ~195 GB → stripe up to 64 URLs |
| **VM snapshot** | Whole VM. Good for ransomware. Not for restoring one database |

Recovery models: **FULL** = point-in-time restore. **SIMPLE** = no log backups, no point-in-time. **BULK_LOGGED** = faster bulk loads, partial point-in-time. Do **not** mix backup tools on one database: it breaks the log chain.

## 8. Top exam traps

- Azure SQL Database has **no SQL Agent**. Use elastic jobs. MI and VM do have Agent.
- MI has **no active geo-replication**. Its cross-region DR is a **failover group**.
- An MI with an **MI link** cannot be in a failover group.
- Geo-replication failover needs a **connection string change**. Failover groups keep the **listener** name.
- Failover group "automatic" (Microsoft managed) waits at least **1 hour** and is for wide outages only. Customer managed is recommended.
- PaaS restore always creates a **new** database. You cannot restore over the old one.
- PITR is **1–35 days**. Anything longer is **LTR**, up to 10 years.
- Deleting the logical server deletes PITR backups, but **LTR backups survive**.
- SQL DB backups **cannot** be restored to SQL Server.
- MI allows only **COPY_ONLY** user backups.
- Geo-restore needs **GRS or GZRS** backup storage. LRS and ZRS cannot geo-restore.
- GRS secondary is **not readable** until failover. RA-GRS is readable all the time.
- TDE is **on by default** in PaaS and must be turned on by you on a VM.
- TDE does **not** hide data from DBAs. **Always Encrypted** does.
- Deterministic Always Encrypted allows equality joins. Randomized is safer but needs **enclaves** for rich queries.
- `DENY` beats `GRANT`. `REVOKE` is neutral.
- Users with `CONTROL` (db_owner) always see **unmasked** data under Dynamic Data Masking.
- Database firewall rules are checked **before** server rules. MI uses **NSGs**, not IP firewall rules.
- **Service endpoint** keeps the public endpoint. **Private Link** removes public exposure.
- Windows authentication for Entra users works on **MI only**, not SQL DB.
- `EXECUTE AS LOGIN` works on SQL Server and MI only. `EXECUTE AS USER` works everywhere.
- Dynamic SQL (`sp_executesql`) breaks **ownership chaining**. The user needs direct rights.
- Automatic tuning needs Query Store **ON + READ_WRITE**.
- Query Store read-only = it is **full**. "Write pressure interval" = **data flush interval**.
- Query Performance Insight is **SQL DB only**, not MI.
- In SQL DB use `sys.dm_db_wait_stats` and `sys.dm_db_resource_stats` (15 s, 1 h). `sys.resource_stats` is 5-min and lives in `master`.
- MAXDOP in SQL DB is a **database scoped configuration**, not `sp_configure`. New SQL DB default is MAXDOP 8.
- `CXCONSUMER` waits are normal. Tune `CXPACKET`.
- `PAGELATCH` = memory page contention (often tempdb). `PAGEIOLATCH` = slow disk.
- RCSI is **on by default** in SQL DB, not in SQL Server or MI.
- Estimated plan does not run the query. Actual plan does.
- BACPAC is **offline**. Over ~150 GB use **SqlPackage**. MI cannot import a BACPAC from the portal.
- Transactional replication is the **only online** migration into SQL DB, and tables need a **primary key**.
- LRS (Log Replay Service) must finish in **30 days** and the target is not readable. MI link has no time limit and is readable.
- DMA (July 2025), Azure Data Studio (28 Feb 2026) and DEA (Dec 2024) are **retired**. Assess with Azure Migrate or Arc.
- Hyperscale can go back to General Purpose only within **45 days**.
- Azure Site Recovery copies whole VMs; it is **not SQL-aware**.
- VM data disks: read cache **on**. Log disks: cache **off**. tempdb on **D:** (wiped on restart).
- Basic Load Balancer is retired. Use **Standard**, a DNN listener, or multiple subnets.
- Elastic job database cannot be **Hyperscale** or **auto-pause serverless**.
