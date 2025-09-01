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

