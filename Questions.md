We need to design the **FactSales** table in **Azure Synapse Analytics dedicated SQL pool** with the following requirements:

* Data will be retained for **five years**.
* Each year, data older than five years must be deleted.
* Data must be **distributed evenly across distributions**.
* Must **minimize deletion time** → use **partitioning** by `OrderDateKey`.

---

### Breakdown of choices:

* **Distribution**:

  * `ROUND_ROBIN` → spreads rows evenly across distributions (✅ balances load, avoids skew).
  * `HASH` → requires a good distribution column (not specified as necessary here).
  * `REPLICATE` → duplicates data on all distributions (bad for a large fact table).
    → Best option: **ROUND\_ROBIN**.

* **Partitioning**:

  * Since we delete **by year**, the partition key should be `OrderDateKey`.
  * This allows us to drop old partitions instead of deleting row by row.

---

### Final Answer:

```sql
WITH CLUSTERED COLUMNSTORE INDEX
DISTRIBUTION = ROUND_ROBIN
PARTITION (OrderDateKey RANGE RIGHT FOR VALUES 
(20170101, 20180101, 20190101, 20200101, 20210101))
```

👉 Correct selections:

* **ROUND\_ROBIN**
* **OrderDateKey**

Let’s break it down:

* The issue: **tempDB contention** identified by **Intelligent Insights** in an **Azure SQL Database**.
* Goal: reduce **contention on tempDB**.

---

### Options analysis:

**A. Implement memory-optimized tables** ✅

* Memory-optimized tables reduce or eliminate tempDB usage (since no need for tempDB for row versioning or intermediate storage).
* This directly helps reduce tempDB contention.
* Supported in Azure SQL Database.

**B. Run `DBCC FLUSHPROCINDB`** ❌

* Clears cached execution plans for a database.
* Doesn’t address tempDB contention at all.

**C. Replace sequential index keys with nonsequential keys** ❌

* This helps reduce latch contention on **indexes** (hotspot issue with identity keys), not **tempDB contention**.

**D. Run `DBCC DBREINDEX`** ❌

* Deprecated command to rebuild indexes (replaced by `ALTER INDEX`).
* Has nothing to do with tempDB contention.

---

👉 Correct Answer:
**A. Implement memory-optimized tables**


Let’s analyze the scenario:

* **Requirement:** Count **new events** in **five-minute intervals**.
* **Important detail:** Must **report only events that arrive during the interval** (no late arrivals, no retractions).
* **Sink:** Delta Lake table.

---

### Output modes in Structured Streaming:

* **Complete** → rewrites the entire result table every trigger. Used for aggregations when you want the whole table. Not efficient here.
* **Append** → only new rows (finalized results) are written to the sink. ✅
* **Update** → only rows that changed since the last trigger are written (good for aggregations with late data).

---

Since the requirement says:

* “**report only events that arrive during the interval**” → meaning finalized counts per window, not updates.
* That matches **Append mode**, because results are appended once the window closes.

---

👉 Correct Answer:
**B. append**

Let’s carefully check the requirements:

* Databases are currently on **SQL Server 2019 (SQL1)**.
* Must be able to **communicate with SQL2 (on-premises)** using **linked server connections**.
* Must **minimize administrative effort**.

---

### Options:

**A. Azure SQL Database** ❌

* PaaS service, but it does **not support linked servers**.

**B. Azure SQL Database elastic pool** ❌

* Same limitations as single Azure SQL DB. No linked servers.

**C. SQL Server on Azure Virtual Machines** ✅ (supports linked servers, since it’s full SQL Server).

* But → requires **high administrative effort** (patching, backups, HA, etc.).

**D. Azure SQL Managed Instance** ✅

* PaaS service with **near 100% SQL Server compatibility**.
* **Supports linked servers** (to on-premises SQL, via private endpoint / VPN / ExpressRoute).
* **Minimizes administrative effort** compared to running SQL on VMs.

---

👉 Correct Answer:
**D. Azure SQL Managed Instance**


We are designing a **star schema** for analytics.
The extract includes **dimensions (descriptive attributes)** and **facts (measurable values)**.

---

### Given columns:

* **EventCategory** → e.g., "Videos" → **dimension**
* **EventAction** → e.g., "Play" → **dimension**
* **EventLabel** → e.g., "Contoso Promotional" → **dimension**
* **ChannelGrouping** → e.g., "Social" → **dimension**
* **TotalEvents** → numeric → **fact**
* **SessionsWithEvent** → numeric → **fact**
* **UsersWithEvent** → numeric → **fact**

---

### Star schema mapping:

1. **EventCategory** → goes to **DimEvent**
   (It’s descriptive about the type of event.)

2. **ChannelGrouping** → goes to **DimChannel**
   (It describes the channel, so belongs to channel dimension.)

3. **TotalEvents** → goes to **FactEvents**
   (It’s a measure/metric, belongs in fact table.)

---

### Final Answer:

* **EventCategory → DimEvent**
* **ChannelGrouping → DimChannel**
* **TotalEvents → FactEvents**


Let’s analyze this step by step:

* **Scenario:**

  * Azure Synapse Analytics (DW1) with **automated data loads**.
  * At the same time, **users run ad-hoc queries**.
  * Requirement: **ensure automated loads have enough memory to finish quickly** despite concurrency.

---

### Options:

**A. Assign a smaller resource class to the automated data load queries** ❌

* Smaller resource class = less memory → opposite of what’s needed.

**B. Create sampled statistics to every column** ❌

* Statistics improve query plans but do not guarantee memory availability for loads.

**C. Assign a larger resource class to the automated data load queries** ✅

* Resource classes in Synapse determine how much memory is allocated to a query.
* Larger resource class = more memory → better for **data load performance**.

**D. Hash distribute the large fact tables before performing the automated data loads** ❌

* Hash distribution helps query performance and reduces data movement, but it does **not** solve the memory availability issue for the loading process.

---

👉 Correct Answer:
**C. Assign a larger resource class to the automated data load queries.**


We need to encrypt **Azure Data Factory v2 (DF1)** using a **customer-managed key (CMK)** from **Azure Key Vault (vault1)**.

---

### Important facts:

* To use **CMK encryption** with ADF, the **Key Vault** must have:

  * **Soft-delete enabled**
  * **Purge protection enabled**

This is a **hard requirement** from Azure because ADF must be able to recover the key if accidentally deleted.

---

### Options:

**A. Disable purge protection on vault1** ❌

* Wrong, purge protection must be **enabled**, not disabled.

**B. Remove the linked service from df1** ❌

* Not required before enabling CMK encryption.

**C. Create a self-hosted integration runtime** ❌

* Not related to encryption with CMK.

**D. Disable soft delete on vault1** ❌

* Wrong, **soft delete must be enabled**, not disabled.

---

👉 The correct first step is actually the **opposite** of A and D:
You must **enable soft-delete and purge protection** on the Key Vault.

Since none of the listed answers say **enable**, and the trick options are about disabling them, the correct interpretation is:

✅ **None of these disabling actions are correct. The required step is to ENABLE soft-delete and purge protection on vault1.**

---


We want an **alert on CPU usage** for **all Azure SQL databases** on the same server, including **future databases**.

---

### Options:

**A. Resource Groups** ❌

* Alerts at resource group scope don’t automatically track database-level metrics like CPU usage.

**B. SQL Servers** ✅

* If you set the alert on the **SQL Server resource**, it applies to all databases under that server, including **new ones created later**.

**C. SQL Databases** ❌

* Works only per individual database. Would require configuring alerts one by one and wouldn’t auto-apply to new databases.

**D. SQL Virtual Machines** ❌

* Not relevant here — we’re using **Azure SQL Database PaaS**, not SQL Server on VMs.

---

👉 Correct Answer:
**B. SQL Servers**

We need **voice notifications** when **maintenance events affect any of the 10 regions** where our **Azure SQL Managed Instances** run.

---

### Options:

**A. From the Azure portal, create a service health alert** ✅

* Service health alerts notify you about **issues in Azure services or regions** (including planned maintenance).
* Can deliver alerts via **email, SMS, or voice call**.
* Configured **once per subscription/region**, so this minimizes administrative effort across all managed instances.

**B. Create an Azure Advisor operational excellence alert** ❌

* Advisor gives best-practice recommendations (cost, performance, security), not service health/maintenance events.

**C. Configure a SQL Server Agent job in SSMS** ❌

* Local to a single managed instance, doesn’t integrate with Azure-wide maintenance event notifications.

**D. Configure an activity log alert** ❌

* Activity log captures events **within your subscription resources** (e.g., resource modifications), not **service health events at the regional level**.

---

👉 Correct Answer:
**A. From the Azure portal, create a service health alert**


Let’s analyze this step by step:

### Requirements:

1. **Failovers that do not require client applications to change their connection strings**
   → This rules out **geo-replication**, because geo-replication requires **manual connection string changes** during failover.

2. **Replicate the database to a secondary Azure region**
   → This means cross-region support is needed (not just Availability Zones, which are within the same region).

3. **Support failover to the secondary region**
   → Must support **automatic or manual failover across regions**.

---

### Options:

* **A. Failover groups** ✅

  * Provides **automatic failover** between **primary and secondary servers** across regions.
  * Applications can connect using a **listener endpoint**, so no connection string changes are required.
  * Meets all requirements.

* **B. Transactional replication** ❌

  * Mainly for replication, not high availability or transparent failover.
  * Requires connection changes and doesn’t provide auto-failover.

* **C. Availability Zones** ❌

  * Increases availability within the **same region**, not across regions.
  * Does not meet the replication-to-secondary-region requirement.

* **D. Geo-replication** ❌

  * Provides readable secondary, but **failover requires manual reconfiguration** of connection strings.
  * Doesn’t meet the requirement.

---

✅ **Correct Answer: A. failover groups**


Great question 👍 Let’s break it down.

You want to see **the parameter values from the last time a query was executed** on **SQL Server running in an Azure VM**.

---

### 🔎 Key SQL Server features involved:

1. **Lightweight Query Profiling**

   * Introduced to reduce overhead of query profiling.
   * Enables capturing runtime execution stats (e.g., row counts, operator performance).
   * Needed for getting **execution stats** in DMVs.

2. **Last\_Query\_Plan\_Stats** (Database Scoped Configuration)

   * Allows you to access the **last actual execution plan** for a query (not just the estimated plan).
   * Includes **runtime parameter values** from the last execution.

3. **PARAMETER\_SNIFFING**

   * Controls whether SQL Server caches query plans based on the first set of parameters.
   * Not related to retrieving parameter values from the last query.

---

### ✅ Correct Answers:

* **B. Enable Lightweight\_Query\_Profiling in DB1**
  (Needed to collect query profiling info with minimal overhead.)

* **C. Enable Last\_Query\_Plan\_Stats in DB1**
  (Lets you retrieve the last actual execution plan, including parameter values.)

---

### ❌ Incorrect options:

* **A & D (Enable in master)** → Wrong scope. These are **database scoped configurations**, not set at master.
* **E (Enable PARAMETER\_SNIFFING)** → Controls plan caching, not parameter value retrieval.

---

✅ Final Answer:
**B. Enable Lightweight\_Query\_Profiling in DB1**
**C. Enable Last\_Query\_Plan\_Stats in DB1**

---


Alright, let’s carefully analyze this one ✅

---

### Requirements:

1. **Support user-initiated backups**

   * Needed for flexibility (restore when you want, not only automated retention).
   * ✅ Available in **SQL Server on Azure VM** (IaaS) → you control SQL Server Agent jobs, backups to Blob Storage, etc.
   * ❌ Azure SQL Database / Managed Instance → backups are automated by Azure (you can restore but not *initiate* backups directly).

2. **Support multiple automatically replicated instances across Azure regions**

   * Means geo-redundancy / high availability.

3. **Minimize administrative effort**

   * Rules out heavy management like Always On AG setup in SQL VMs.

---

### Deployment Option:

👉 **SQL Server on Azure Virtual Machines**

* Because only in IaaS SQL VM can you run **user-initiated backups**.
* PaaS (Azure SQL Database / Managed Instance) = automatic backups only, no user-initiated.

---

### Resiliency Option:

👉 **Auto-failover group**

* Provides **automatic replication across regions** + automatic failover with **no connection string change**.
* Active geo-replication = database-level only (manual failover).
* Zone-redundant deployment = only within the same region, not across regions.

---

### ✅ Correct Answer:

* **Deployment option**: **SQL Server on Azure Virtual Machines**
* **Resiliency option**: **Auto-failover group**


Great, let’s break this one down carefully 👇

---

### **NEW QUESTION 49 (Exam Topic 5)**

You have an Azure SQL Database.
You are reviewing a **slow performing query**. The screenshot shows a query plan with operators.

---

### **Step 1 – Identify what the exhibit shows**

* In the query plan screenshot, the header indicates "Query cost relative to the batch: 100%".
* The boxes show query execution operators with percentages.
* This is a **visual execution plan** that SQL Server produces when you run queries.
* Since it shows **estimated vs actual execution details** (with percentages), this is an **actual execution plan**.

👉 **Answer:** **an actual execution plan**

---

### **Step 2 – Identify the operator in the plan**

Looking at the exhibit:

* The bottom operator is labeled **Clustered Index Scan**.
* But the question asks for the **operator in the execution plan that indicates the query would benefit from tuning**.
* The operator causing performance issues here is **Key Lookup (Clustered)** (since key lookups repeatedly access rows from a clustered index — expensive for performance).

👉 **Answer:** **Key Lookup**

---

### ✅ Final Answer:

1. The exhibit shows → **an actual execution plan**
2. The operator in the execution plan that indicates query would benefit from performance tuning → **Key Lookup**

---

⚡ Tip: In SQL exams, whenever you see **Key Lookup** or **Index Scan** with high cost %, it usually means missing indexes → performance tuning required.

Good question 👇

We need anomaly detection for **streaming IoT hub data** with **minimum development effort**. Let’s evaluate:

---

### **Requirements**

* **Send output to Azure Synapse** → must support Synapse sink integration.
* **Identify spikes and dips in time series** → anomaly detection on real-time streaming data.
* **Minimize development/configuration effort** → prefer serverless, built-in features over heavy coding.

---

### **Option Analysis**

**A. Azure SQL Database**

* Not suitable for anomaly detection on streaming data.
* It’s more for storage, not real-time analytics.
  ❌

**B. Azure Databricks**

* Powerful for machine learning and advanced analytics.
* But requires **custom ML models**, coding, and more configuration.
* Does not minimize effort.
  ❌

**C. Azure Stream Analytics**

* Has **built-in anomaly detection functions** (e.g., `AnomalyDetection_SpikeAndDip`).
* Natively integrates with **Azure IoT Hub (input)** and **Azure Synapse Analytics (output sink)**.
* **Serverless, minimal setup**, exactly fits requirements.
  ✅

---

### ✅ Correct Answer:

**C. Azure Stream Analytics**

---

Great one 👇 This is about ensuring your **Azure Stream Analytics job has enough Streaming Units (SUs)** provisioned.

---

### **Key Metric: SU % Utilization**

* This shows how much of your allocated Streaming Unit capacity is being used.
* If it's close to 100%, you may need to scale up.

But monitoring SU utilization alone is not enough. You need to check if the job is keeping up with the input data stream.

---

### **Options Analysis**

**A. Late Input Events**

* These occur when events arrive later than the configured *late arrival tolerance*.
* They’re about data quality, not SU capacity.
  ❌

**B. Out of order Events**

* These are events that arrive out of sequence.
* Also about event ordering, not about whether SUs are sufficient.
  ❌

**C. Backlogged Input Events**

* Very important.
* Shows how many input events the job has received but not yet processed.
* If backlog grows → SUs are insufficient.
  ✅

**D. Watermark Delay**

* Indicates how far the system is behind in processing events, measured in time.
* If watermark delay is increasing → job is under-provisioned.
  ✅

**E. Function Events**

* Refers to events sent to Azure Functions (custom processing).
* Not relevant to SU sizing.
  ❌

---

### ✅ Correct Answers:

**C. Backlogged Input Events**
**D. Watermark Delay**

---


### **Question Recap**

* DB: **Azure SQL database (db1)**
* Monitoring tool: **Query Performance Insight (QPI)** → relies on **Query Store**.
* Requirement: **Performance monitoring data must be available as soon as possible**.
* Task: Choose **configuration setting** and **value**.

---

### **Understanding Query Store Configurations**

1. **DATA\_FLUSH\_INTERVAL\_SECONDS**

   * Controls how often collected data is written (“flushed”) from memory to disk.
   * **Default = 900s (15 min)**.
   * Lowering this value (e.g., 60s or 1s) makes data available faster in QPI.

2. **INTERVAL\_LENGTH\_MINUTES**

   * Defines time granularity of data aggregation (default = 60 min).
   * Smaller intervals (e.g., 1 min) improve granularity but don’t directly affect how soon data is available.

3. **MAX\_PLANS\_PER\_QUERY**

   * Controls how many execution plans are stored → irrelevant for timeliness.

4. **QUERY\_CAPTURE\_MODE**

   * Controls which queries are captured (ALL, AUTO, CUSTOM).
   * Impacts **what is captured**, not **when it is available**.

---

### **Correct Answer**

To make **performance monitoring data available ASAP**, you must:

* Configure **DATA\_FLUSH\_INTERVAL\_SECONDS**
* Set to **1 (second)**

---

✅ **Final Answer:**
**Configuration setting:** `DATA_FLUSH_INTERVAL_SECONDS`
**Value:** `1`

---

### Command structure:

```sql
RESTORE DATABASE MyDB1
FROM DISK = 'D:\DB1.bak'
WITH { option }
GO
```

---

### Options:

1. **NORECOVERY**

   * Leaves the database **non-operational**.
   * Used when you plan to apply **additional transaction log backups**.
   * Keeps the database in a "restoring" state.
   * Example:

     ```sql
     RESTORE DATABASE MyDB1 
     FROM DISK = 'D:\DB1.bak' 
     WITH NORECOVERY;
     ```

2. **RECOVERY**

   * Makes the database **operational** immediately after the restore.
   * You cannot restore more backups after this.
   * Example:

     ```sql
     RESTORE DATABASE MyDB1 
     FROM DISK = 'D:\DB1.bak' 
     WITH RECOVERY;
     ```

3. **STANDBY**

   * Makes the database **read-only** after restore.
   * Useful for log shipping (secondary database is readable while waiting for the next log restore).
   * Example:

     ```sql
     RESTORE DATABASE MyDB1 
     FROM DISK = 'D:\DB1.bak' 
     WITH STANDBY = 'D:\UNDO\DB1.undo';
     ```

---

✅ **In short:**

* Use **NORECOVERY** if more restores are coming (like log backups).
* Use **RECOVERY** if this is the final restore and you want the DB online.
* Use **STANDBY** if you want a read-only copy (common in log shipping).


### Key point in the question:

* Customers need to be able to **create database objects**.
* The solution must meet **business goals** → usually means **isolation** between customers, avoiding conflicts.

---

### Roles:

* **`ddl_admin`**: Allows a user to run **Data Definition Language (DDL)** commands such as `CREATE`, `ALTER`, `DROP`.
* **`db_writer`**: Only allows `INSERT`, `UPDATE`, `DELETE` data → **cannot create schema objects**.

---

### Options:

**A. Grant `ddl_admin` on existing schema**

* Problem: All customers share the same schema → conflicts, no isolation. ❌

**B. Create additional schema per customer and grant `ddl_admin`**

* ✅ Best option.
* Each customer gets their own schema, and `ddl_admin` on that schema allows them to create their own tables, procedures, etc.
* Prevents conflicts, aligns with multi-tenant isolation.

**C. Create additional schema and grant `db_writer`**

* ❌ `db_writer` only allows DML, not schema changes.

**D. Grant `db_writer` on existing schema**

* ❌ Same as above, plus shared schema (bad for isolation).

---

✅ **Correct Answer: B. For each customer, create an additional schema and grant the customer ddl\_admin to the new schema.**

---

### Scenario:

* You need to implement **security** for **ResearchDB1**.
* Requirement → must meet **security and compliance**.
* Clearly, this is about **Always Encrypted** with **Azure Key Vault** integration.

---

### Steps (in correct sequence):

1. **Create an Azure AD managed identity**

   * Needed so ResearchApp1 can authenticate securely without storing credentials.

2. **Create an Azure Key Vault instance and configure an access policy**

   * Key Vault stores encryption keys.
   * Access policy ensures the managed identity has permission to use the keys.

3. **Run the Always Encrypted wizard**

   * This wizard configures column encryption and points to the Key Vault for key management.

---

### Answer:

✅ Correct sequence is:

1. **Create an Azure AD managed identity**
2. **Create an Azure Key Vault instance and configure an access policy**
3. **Run the Always Encrypted wizard**

---

⚡Pro tip:
The option **“Create an Azure Key Vault instance and generate a secret”** is a distractor. For Always Encrypted, you store **keys**, not secrets.

---


Good question 👌 Let’s analyze it.

---

### Scenario:

* You have **an Azure SQL Database server** (`sqlsrv1`) with 10 databases.
* Performance is slower than expected.
* You specifically want to check **if the issue is related to `tempdb`**.

---

### Options:

**A. Run Query Store-based queries**

* Query Store helps with **query performance** over time (execution plans, regressions).
* Not about `tempdb`. ❌

**B. Review information provided by SQL Server Profiler-based traces**

* SQL Profiler is not supported for Azure SQL Database. ❌

**C. Review information provided by Query Performance Insight**

* Gives insights into **top consuming queries** (DTU, CPU, IO, memory).
* Doesn’t specifically help diagnose `tempdb` usage. ❌

**D. Run dynamic management view (DMV)-based queries**

* DMVs in Azure SQL Database can show resource usage, waits, and specifically **tempdb contention** (e.g., `sys.dm_db_resource_stats`, `sys.dm_exec_requests`, `sys.dm_db_session_space_usage`).
* This is the correct way to check `tempdb` bottlenecks. ✅

---

### ✅ Correct Answer:

**D. Run dynamic management view-based queries**

---

Great question 👍 Let’s break it down clearly:

---

### **Why “Simple layout (striping)” is the correct choice here**

* You have **two P30 disks**, each provides **5,000 IOPS**.
* Requirement: **10,000 IOPS**.
* The **simple layout** (striping) combines both disks’ throughput and IOPS, giving you **aggregate performance** (5,000 + 5,000 = 10,000 IOPS).
* This meets the requirement.

---

### **Other layouts and when they are used**

1. **Mirror layout**

   * Writes are duplicated across disks.
   * You don’t get combined IOPS; instead, you get **redundancy**.
   * Example: If you had two disks with 5,000 IOPS each, effective IOPS remains 5,000.
   * Use case: **High availability and redundancy** (at the cost of performance/space).

2. **Parity layout**

   * Data + parity bits are striped across disks.
   * Provides fault tolerance with better capacity than mirror, but **slower writes**.
   * Use case: **Archival or workloads that need balance between capacity and resiliency**.
   * (Not shown in your options, but good to know.)

3. **Simple layout (striping)**

   * Data is striped across disks, **no redundancy**.
   * All disks contribute to performance (IOPS + throughput).
   * Use case: **High-performance workloads where redundancy is not critical** (e.g., tempdb, non-critical analytical workloads, or when redundancy is provided elsewhere).

4. **Storage Pool**

   * Logical grouping of multiple physical disks.
   * Foundation step before creating virtual disks with layouts.
   * Use case: Always the **first step** when using Storage Spaces in Windows/Azure VM.

---

✅ In your scenario:

* Performance (10,000 IOPS) is the **primary requirement**.
* Redundancy is **not mentioned**.
* So → **Storage Pool → Simple Layout → Volume** is correct.

---


Correct ✅

The answer is: **B. Decrease by half the value of Data Flush Interval**

### Explanation:

* Query Store collects query performance data in memory first, then periodically flushes it to disk (the database).
* If the **Data Flush Interval** is too long, Query Store can accumulate too much data in memory. If SQL Server runs out of space for Query Store, it can transition into **read-only mode** to prevent further growth.
* By **decreasing the Data Flush Interval**, data is written to disk more frequently, reducing memory pressure and minimizing the risk of Query Store becoming read-only.

Other options:

* **A. Double Data Flush Interval** → Increases risk (keeps more data in memory).
* **C & D. Statistics Collection Interval** → Controls how often runtime statistics are aggregated, not related to Query Store read-only transitions.

👉 So the best solution is **B**.

Perfect ✅ let’s break this down step by step using the policy in the screenshot.

### **Retention Policy in Screenshot**

* **Weekly LTR Backups** → Retained for **6 weeks**.
* **Monthly LTR Backups** → Retained for **12 months** (the **Week 2 backup** of each month).
* **Yearly LTR Backups** → Retained for **10 years** (the **Week 2 backup of January** each year).

---

### **Example: January 2020**

1. **Jan 4, 2020** → Week 1 of Jan

   * Stored as **weekly backup**.
   * Retained **6 weeks** only.
   * ❌ Not monthly, because policy says Week 2.

2. **Jan 11, 2020** → Week 2 of Jan

   * Stored as **weekly backup** → retained **6 weeks**.
   * ✅ Promoted to **monthly backup** → retained **12 months**.
   * ✅ Also promoted to **yearly backup** (since it’s the Week 2 backup of January) → retained **10 years**.

---

### **February 2020**

* **Feb 1, 2020** → Week 1 → weekly only → 6 weeks.
* **Feb 8, 2020** → Week 2 → weekly + promoted to monthly → retained **12 months**.

(no yearly in February, only January provides yearly).

---

### **Summary**

* **Weekly backups**: Every week, kept **6 weeks**.
* **Monthly backups**: The **Week 2 backup** of each month, kept **12 months**.
* **Yearly backups**: The **Week 2 backup of January**, kept **10 years**.

---

👉 So the reason **Jan 4, 2020** is **not** a monthly backup is simply because the **monthly rule is configured for Week 2**, not Week 1.


You got it 👍

For **triggering an Azure Data Factory pipeline when a file arrives in Azure Data Lake Storage Gen2**, the correct service is **Event Grid**.

* **Microsoft.EventHub** → Used for streaming large volumes of events (real-time ingestion).
* **Microsoft.Sql** → SQL resource provider (not relevant to file triggers).
* **Microsoft.Automation** → Used for automation runbooks, not event-driven triggers.
* ✅ **Microsoft.EventGrid** → Correct. Event Grid integrates with **ADLS Gen2** and can raise an event when a new file is created, which can then trigger the **ADF pipeline**.

**Final Answer: B. Microsoft.EventGrid** ✔️

Great question — this comes up a lot in Synapse design ✅

You’re creating a **dimension table** in a **dedicated SQL pool** and need a **surrogate key** with **fast query performance**.

Let’s analyze:

* **A. an IDENTITY column**

  * Common in OLTP systems (SQL Server).
  * But in Synapse dedicated SQL pool, `IDENTITY` values are generated per distribution, which can lead to **duplicate values across distributions** unless carefully managed. Not the best for scalability or performance in MPP (Massively Parallel Processing).

* **B. a GUID column**

  * Globally unique, but **large (16 bytes)** and not sequential.
  * Leads to fragmentation and **slower query performance**. Bad choice for surrogate keys in Synapse.

* **C. a sequence object**

  * ✅ Recommended in Synapse.
  * Centralized and generates unique values across distributions.
  * Provides **scalable, fast, and sequential surrogate keys** — avoids the pitfalls of `IDENTITY`.

📌 **Correct Answer: C. a sequence object**

Great question 👍 Let’s break down why the **other options** are not correct for this specific requirement and when you *would* use them:

---

### **Option A. Attach two mirrored 4-TB SSDs (RAID-1 / Mirroring)**

* **Why not here?**

  * Mirroring cuts usable capacity in half (4 TB usable from 8 TB raw).
  * IOPS does **not scale** much compared to striping.
  * Goal here is **maximum IOPS**, not redundancy.
* **When to use:**

  * If your main requirement is **high availability** or **redundancy**, e.g., protecting against a single disk failure.
  * Not ideal for Azure Premium SSDs since Azure already provides redundancy at the infrastructure level.

---

### **Option C. Attach a RAID-5 array with five 1-TB SSDs**

* **Why not here?**

  * RAID-5 has **write penalties** (parity calculation) → not good for SQL Server workloads with high write activity.
  * More complex setup and slower performance compared to striping (RAID-0).
* **When to use:**

  * Suitable when you want a **balance between capacity and redundancy** and workloads are **read-heavy** (but rarely used for SQL Server on Azure due to performance trade-offs).

---

### **Option D. Attach a single 4-TB SSD**

* **Why not here?**

  * A single disk’s IOPS and throughput are capped by Azure limits for that disk size.
  * Even the largest Premium SSD can’t reach the IOPS you get by **striping multiple smaller SSDs**.
* **When to use:**

  * If your requirement is **simplicity** (easy to manage, one disk).
  * If workload IOPS needs fit within the single-disk cap.

---

### ✅ **Correct Choice: Option B. Stripe set with four 1-TB SSDs (RAID-0 / Storage Spaces)**

* Premium SSDs in Azure **scale IOPS linearly when striped**.
* Striping provides the **highest throughput and IOPS**.
* Perfect when the requirement is **performance** over redundancy.

---

⚖️ **Summary:**

* Use **Striping (RAID-0)** → for maximum IOPS/throughput.
* Use **Mirroring (RAID-1)** → for redundancy/high availability.
* Use **RAID-5** → for capacity efficiency + some fault tolerance (but rarely in SQL Server Azure due to poor write performance).
* Use **Single large SSD** → for simplicity when IOPS needs are moderate.

---

Great question 👍 Let’s break down when each option (distribution + index) can be used:

---

### **Distribution types**

1. **Hash**
   ✅ Best for: **Fact tables** (large tables, high number of rows).

   * Use when you can choose a **high-cardinality column** as the distribution key (like CustomerID, OrderID).
   * Optimizes joins and aggregations.

2. **Round robin**
   ✅ Best for: **Staging tables** or when no good distribution key exists.

   * Data is evenly distributed, but not based on a column → can lead to more data movement during joins.
   * Example: Landing raw data before transformations.

3. **Replicated**
   ✅ Best for: **Small dimension tables** (usually < 2 GB compressed).

   * A full copy of the table is stored on every compute node.
   * Eliminates data movement in joins between fact and dimension tables.
   * Example: Date dimension, Country lookup.

---

### **Index types**

1. **Clustered columnstore index (CCI)**
   ✅ Best for: **Fact tables in analytics/OLAP**.

   * Provides high compression + very fast scans for aggregates.
   * Default for large fact tables in Synapse.

2. **Clustered index**
   ✅ Best for: **Small tables or OLTP-like workloads**.

   * If you need fast point lookups (e.g., get 1 record by key).
   * Sometimes used on small dimension tables instead of CCI (to avoid compression overhead).

3. **Nonclustered index**
   ✅ Best for: **Improving specific query patterns**.

   * Example: When you need to optimize filtering on a few columns.
   * Usually applied on top of CCI for targeted queries (but use sparingly to avoid load overhead).

---

✅ **Summary rule of thumb:**

* **Fact tables (big, >1M rows):** Hash + CCI.
* **Small dimension tables (<2GB):** Replicated + Clustered or CCI.
* **Staging tables:** Round robin + Clustered index.
* **Special queries:** Add Nonclustered index as needed.

---

Let’s break this one down carefully:

---

### ✅ Requirements:

* **App1 authenticates with SQL authentication** → means username/password inside DB, not Azure AD or Windows login.
* **App1 must only view DB2** → should not see DB1.
* **Minimal administrative effort** → simplest, direct solution.

---

### 🔎 Options:

**A. a contained database user for App1 on DB2**

* A contained database user is **created inside the database only** (DB2).
* It doesn’t require a server-level login on `Server1`.
* This means App1 only sees **DB2** and not DB1.
* ✅ This meets **all requirements**.

---

**B. a login for App1 on Server1**

* A server-level login would exist at `Server1`.
* By default, it has visibility to the server and could be mapped to both DB1 and DB2.
* ❌ More admin overhead, doesn’t restrict visibility to DB2 only without extra steps.

---

**C. a contained database user from an external provider for App1 on DB2**

* "External provider" = Azure AD authentication.
* ❌ Doesn’t apply here since App1 is using **SQL authentication**, not Azure AD.

---

**D. a contained database user from a Windows login for App1 on DB2**

* This is Windows authentication.
* ❌ Not valid, since App1 must authenticate via **SQL authentication**.

---

### ✅ Correct Answer:

**A. a contained database user for App1 on DB2**

---


Perfect, let’s analyze this one 👇

The query execution plan shows a **Key Lookup (Clustered)** operator.

### Breakdown:

* **Key Lookup** happens when SQL Server finds a row in a **nonclustered index**, but needs more columns not present in the index, so it goes back to the **clustered index (or heap)** to fetch them.
* This causes extra I/O and slows performance.
* The fix is to add those missing columns into the **nonclustered index** as **INCLUDE columns**, so SQL Server doesn’t need to do the lookup.

### Correct Answer:

* **The performance issue stems from the:** 👉 **Key Lookup** operator
* **The performance issue can be resolved by adding include columns to the:** 👉 **nonclustered index**

✅ Final Answer:

* **Key Lookup**
* **nonclustered index**

Great, let’s work through this one step by step 👇

### Scenario:

* **DB1** = Azure SQL Database (Hyperscale tier, no secondary replicas yet).
* **App1** = Azure Web Apps (multiple instances, read-only access to DB1).
* Requirement: Create a **read-only replica** of DB1 and configure **App1** to use it.

---

### Key Concepts:

1. **Hyperscale tier** supports **read-scale replicas** (read-only replicas).

   * You can create **1–4 named replicas** on the same logical server.
   * These replicas can be used for **read-only workloads** like reporting or queries.

2. To direct apps to replicas:

   * Use the **ApplicationIntent=ReadOnly** parameter in the connection string.
   * This ensures the app routes queries to the replica, not the primary.

---

### Correct Steps:

* **To add read-only replicas of DB1** → ✅ *Create a replica on the same logical server.*
* **To configure App1 instances to access the read-only replica** → ✅ *Add an ApplicationIntent entry to the connection string.*

---

✅ **Final Answer:**

* **Create a replica on the same logical server**
* **Add an ApplicationIntent entry to the connection string**

---

