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


