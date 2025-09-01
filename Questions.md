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

