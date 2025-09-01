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
