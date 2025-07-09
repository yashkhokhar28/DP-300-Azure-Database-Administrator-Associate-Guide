## 🔍 **What Is Query Store?**

Query Store is a **built-in feature in SQL Server and Azure SQL Database** that helps you:

- **Track query performance over time.**
- **Store query plans**, runtime statistics, and wait stats.
- **Fix performance problems** when queries suddenly run slower.

Without Query Store, SQL Server only keeps **the latest query plan in memory**, and older plans disappear when memory is reused.
With Query Store, **you have a permanent history of query performance and plans**, even after restarts.

---

## 📂 **Query Store Has 3 Key Stores:**

| Store                   | Purpose                                                        |
| ----------------------- | -------------------------------------------------------------- |
| **Plan Store**          | Saves all execution plans (estimated + forced plans).          |
| **Runtime Stats Store** | Saves query execution stats (CPU, duration, row counts, etc.). |
| **Wait Stats Store**    | Tracks the wait times (e.g., I/O waits, lock waits).           |

---

## ⚙️ **How Query Store Works**

- Query Store **integrates into the query execution pipeline**.
- It collects and saves data when queries:

  - **Run for the first time** → query text + estimated plan saved.
  - **Recompile** → new plans are saved too.
  - **Execute repeatedly** → collects runtime stats like CPU, duration, etc.

All of this is written to disk **asynchronously**, meaning it won’t slow down your queries too much.

---

## ✅ **How to Enable Query Store**

In **Azure SQL Database**, it's **enabled by default**.
In SQL Server or Synapse Analytics, turn it on like this:

```sql
ALTER DATABASE <database_name> SET QUERY_STORE = ON (OPERATION_MODE = READ_WRITE);
```

---

## 🔑 **Common Use Cases**

| Scenario                | Example                                                                  |
| ----------------------- | ------------------------------------------------------------------------ |
| Fix regressions         | A query used to run in 1 sec but now takes 10 sec after an index change. |
| Find top resource hogs  | Identify queries using the most CPU, memory, or causing waits.           |
| A/B testing             | Measure the performance difference between two versions of a query.      |
| Keep performance stable | Lock a good execution plan in place during upgrades.                     |
| Analyze workloads       | See your database's query patterns over time.                            |

---

## 🔬 **Key Query Store Views in SSMS**

Once enabled, you can see these reports in SQL Server Management Studio (**SSMS**) → Database → Query Store:

### 1. **Regressed Queries**

- Finds queries whose **performance got worse over time**.
- Shows **old vs new plans**, lets you **force the older, faster plan**.
- You can also force a plan with T-SQL:

```sql
EXEC sp_query_store_force_plan @query_id = 73, @plan_id = 79;
```

### 2. **Overall Resource Consumption**

- Graphs of **total resource usage** (CPU, duration, etc.).
- Lets you drill down to the **queries causing the load**.

### 3. **Top Resource Consuming Queries**

- Quickly shows the **top 25 most expensive queries**.
- Helps identify heavy queries, even if they only ran once.

### 4. **Queries With Forced Plans**

- Shows which queries have **manually forced plans**.
- Lets you unforce them if the forced plan is no longer best.

### 5. **Queries With High Variation**

- Lists queries with **inconsistent performance**.
- Useful for spotting queries that are sometimes fast, sometimes slow.

### 6. **Query Wait Statistics**

- Analyzes **wait types** like I/O, locks, CPU.
- Lets you drill down to **the queries causing those waits**.

---

## 🤖 **Automatic Plan Correction**

SQL Server 2017+ and Azure SQL Database can use Query Store data to **automatically fix slow queries.**

- SQL watches for **query plan regressions** and recommends forcing a better plan.
- If you trust it, you can enable **automatic plan fixing**:

```sql
ALTER DATABASE <database_name> SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON);
```

In Azure SQL, it's **enabled by default on new databases**.

---

## 🛡️ **Bonus Tools**

- **Track Specific Query:** Follow the performance history of a query by its ID.
- **sys.dm_db_tuning_recommendations:** Shows **plan correction suggestions** from SQL Server.

---

## ✅ **Summary**

| Purpose                     | Query Store Helps You        |
| --------------------------- | ---------------------------- |
| Query Tuning                | Find and fix slow queries    |
| Plan History                | Keep track of all past plans |
| Performance Troubleshooting | Identify causes of slowdowns |
| Automatic Tuning            | Let SQL auto-fix regressions |

---
