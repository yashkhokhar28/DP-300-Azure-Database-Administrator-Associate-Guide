# Configure compute, storage and server settings

> **Exam skills:** Configure server settings for performance, Configure Resource Governor for performance, Configure compute and storage resources for scaling
> **Reading time:** ~12 min

## The big idea
A database needs enough **compute** (CPU and memory) and fast enough **storage** (disks). Too little and it is slow. Too much and you waste money. On an Azure VM you choose disks, VM size and server settings yourself. In Azure SQL Database and Managed Instance, Azure does much of this, and you mostly choose a tier and size.

Think of a restaurant kitchen. The chefs are the CPUs, the worktop is memory, and the fridge is storage. A fast chef with a tiny worktop and a slow fridge still serves food slowly.

## Storage for SQL Server on Azure VMs
Use **managed disks** for data and log files, **Blob storage** for backups ("backup to URL"), and **Azure Files** for shared storage in failover cluster instances.

### Managed disk types
| Disk | Best for | Notes |
|------|----------|-------|
| **Ultra Disk** | Most critical workloads | Very high IOPS (input/output operations per second), under 1 ms latency |
| **Premium SSD v2** | Most modern production data and log | Sub-ms latency; size, IOPS and throughput set separately; usually cheaper than Ultra |
| **Premium SSD** | Production | Good speed and price |
| **Standard SSD** | Dev/test | Medium speed |
| **Standard HDD** | Dev/test, archive | Slowest |

For production use **Premium SSD v2, Premium SSD or Ultra Disk**. Premium SSD v2 is recommended with the **Ebdsv5** VM series.

### Disk best practices
- Put **data and log files on separate disks**.
- **Data disks:** turn on **read-only host caching** (faster reads). This works on **Premium SSD only**. Premium SSD v2 and Ultra Disk do not support host caching, and neither do disks of 4 TiB or more. Stop SQL Server before changing cache settings.
- **Log disks:** set caching to **None**.
- Put **tempdb on the local D: drive** (temporary local SSD). It is wiped on reboot, but tempdb is rebuilt at every start anyway.
- Turn on **Instant File Initialization** (data files grow faster because SQL Server skips writing zeros).
- Format data disks with a **64-KB allocation unit size**.
- Plan about **20% extra IOPS and throughput** for spikes.
- On **M-series** VMs, use **Write Accelerator** for the **log** disk (never for data).

**Disk striping:** combine several disks into one volume with **Storage Spaces** (Windows) to add up their IOPS and size. Do **not** turn on redundancy in the pool; Azure already keeps copies.

**Example:** 10 × 1 TB Premium SSDs at 5,000 IOPS each, striped together = one 10 TB volume with **50,000 IOPS**.

## Choosing and resizing the VM (compute)
SQL Server is licensed **per core**, so more cores = higher licence cost. Memory and IOPS often matter more than CPU.

**VM families:**
- **Memory-optimized** sizes are recommended for production: a memory-to-vCore ratio of **8:1** and at least **4 vCores**.
- **Ebdsv5** is the recommended starting point (best price-performance).
- **Mbdsv3 / M-series** for mission-critical or the largest workloads.
- **D-series and B-series** (general purpose) are for dev/test.

**Constrained cores:** a VM size that keeps all the memory, storage and I/O of a big VM but exposes only **½ to ¼** of its vCPUs (for example **M64-32ms**, **M64-16ms**). Compute cost stays the same, but you pay SQL Server licences for fewer cores.

**Example:** A reporting database needs 512 GB of memory but only light CPU. Instead of licensing 64 cores, the DBA picks a constrained-core size with 16 active cores and the same memory. The licence bill drops a lot.

**Resizing a VM:**
- You can resize any time as the workload grows or shrinks.
- Azure **restarts** the VM, so expect a **short outage**.
- Sometimes you must **deallocate** (fully stop) the VM first, for example when moving to a different VM family.

## Storage in Azure SQL Database and Managed Instance
- You do not manage disks. Pick a **service tier** (a level of service with its own speed and price).
- For better IOPS and lower latency, use **Business Critical** or **Hyperscale**.
- **Azure SQL Database** has only **one data file** (Hyperscale usually several). You cannot add files.
- **Managed Instance** lets you add files and change file sizes, but not choose physical disks.

### tempdb
**tempdb** is a shared system database for temp tables, sorts, temporary results and row versions.
| Platform | tempdb files |
|----------|--------------|
| **SQL Server** | Setup creates one file per logical processor, **up to 8**. Keep sizes equal. If contention stays, add files in **multiples of 4** (up to the number of cores). |
| **Azure SQL Database** | Number of files grows with vCores (e.g. 2 vCores = 4 files), **max 16**. You cannot change it. |
| **Managed Instance** | **12 data files** by default. You **can** change the number (max 128), growth and max size. Settings survive restarts and failovers. |

Memory-optimized tempdb metadata (SQL Server 2019+) is **not** available in Azure SQL Database or Managed Instance.

## Scaling compute and storage in PaaS
- **Scale up/down:** change the vCores or service tier. Use this when CPU, memory, I/O or log rate stays at the limit.
- **Hyperscale** separates compute from storage:
  - Storage grows automatically up to **128 TB** per database.
  - Compute can be scaled on its own.
  - Add **read replicas**: up to **4 HA replicas**, up to **30 named replicas**, plus geo-replicas. Send read-only work there with `ApplicationIntent=ReadOnly`.
  - Moving to Hyperscale is one-way, except you can go back to **General Purpose within 45 days**.

**Example:** A news site's database grows 5 TB a year. In **Hyperscale**, storage grows on its own and named replicas serve the reporting team.

## Server settings for performance
These are mainly **instance-level** settings on SQL Server (VM) and Managed Instance, set with `sp_configure`.

| Setting | What it does | Tip |
|---------|--------------|-----|
| **max degree of parallelism (MAXDOP)** | How many CPU cores one query may use at once | Too high = more memory use and `CXPACKET` waits on busy systems |
| **cost threshold for parallelism** | How "expensive" a query must be before SQL Server runs it in parallel (default 5) | Raise it so small queries stay single-threaded |
| **max server memory** | Upper limit of memory SQL Server takes | Set it so the OS keeps enough memory |
| **optimize for ad hoc workloads** | Stores only a small stub for a query plan the first time a query runs | Saves memory when many queries run only once |

```sql
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'max degree of parallelism', 4; RECONFIGURE;
EXEC sp_configure 'cost threshold for parallelism', 50; RECONFIGURE;
```

**In Azure SQL Database** you cannot use `sp_configure`. Set MAXDOP per database instead:
```sql
ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = 4;
```
New Azure SQL databases default to **MAXDOP 8**. You can also use the query hint `OPTION (MAXDOP n)` for one query.

**Example:** An order system on a 16-core VM shows many `CXPACKET` waits. Small queries are going parallel and fighting for CPU. The DBA sets MAXDOP to 8 and raises the cost threshold for parallelism, so only big reports use many cores.

## Resource Governor
**Resource Governor** limits how much **CPU, memory and I/O** each group of sessions can use. It stops one workload from starving others.
- Available in **SQL Server** and **Azure SQL Managed Instance**.
- **Not available in Azure SQL Database** (the platform manages resources).
- **SQL Server 2025:** now also in **Standard** edition (before: Enterprise/Developer only) and can limit **tempdb space** per workload group.

### Three parts
1. **Resource pool** – a slice of server resources.
   - **Internal** pool: for SQL Server itself; cannot be limited.
   - **Default** pool: where sessions go if not classified.
   - **User-defined** pools: you create them.
   - Settings: MIN/MAX CPU %, **CAP_CPU_PERCENT** (a hard cap, used even when the server is idle), MIN/MAX memory %, MIN/MAX IOPS per volume, NUMA affinity.
2. **Workload group** – a group of sessions. Each group belongs to **one pool**; a pool can have many groups.
3. **Classifier function** – a user function that runs **at login** and returns the workload group name for the new session. Keep it fast, because it runs on every connection.

```sql
CREATE FUNCTION dbo.RGClassifier() RETURNS sysname WITH SCHEMABINDING
AS BEGIN
  IF SUSER_NAME() = 'ReportUser' RETURN 'ReportGroup';
  RETURN 'default';
END;
GO
ALTER RESOURCE GOVERNOR WITH (CLASSIFIER_FUNCTION = dbo.RGClassifier);
ALTER RESOURCE GOVERNOR RECONFIGURE;
```

- Changes apply to **new sessions** only.
- On **Managed Instance**, the configuration lives in `master` and **replicates to secondary replicas**. In SQL Server availability groups you must configure **each replica**.

**Example:** One SQL Server runs a shop's online orders and its reports. Reports log in as `ReportUser`. The DBA puts them in a pool capped at 30% CPU. Checkout stays fast even when a big report runs at lunchtime.

## Remember for the exam
- Production disks: **Premium SSD v2, Premium SSD or Ultra Disk**. Standard = dev/test.
- **Read-only caching on data disks, None on log disks.** Host caching only on Premium SSD (not v2, not Ultra, not 4 TiB+).
- **tempdb on the local D: drive** on a VM.
- **Constrained cores** = same memory and I/O, fewer cores, lower SQL licence cost.
- VM resize = **restart** (short outage); a family change may need **deallocate**.
- tempdb files: SQL Server up to 8 then multiples of 4; SQL DB fixed (max 16); MI 12 by default and changeable.
- **cost threshold for parallelism** controls *when* a query goes parallel; **MAXDOP** controls *how many* cores.
- Azure SQL Database: no `sp_configure`; use **ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP**.
- **Resource Governor: SQL Server and MI only**, classifier runs at login, internal pool cannot be limited.
- **CAP_CPU_PERCENT** is a hard limit; MAX_CPU_PERCENT only applies when there is contention.

## Quick check
1. A memory-heavy, CPU-light workload on an Azure VM has high SQL Server licence costs. What do you choose?
   <details><summary>Answer</summary>A constrained-core VM size. It keeps the memory and I/O but exposes fewer cores to license.</details>
2. You must stop a reporting login from using more than 20% CPU on Azure SQL Managed Instance, even when the server is idle. What do you use?
   <details><summary>Answer</summary>Resource Governor with a resource pool using CAP_CPU_PERCENT = 20 and a classifier function that sends the login to that pool's workload group.</details>
3. Which caching setting do you use for a Premium SSD that holds the transaction log?
   <details><summary>Answer</summary>None. Log writes must go straight to disk; read-only caching is for data disks.</details>
