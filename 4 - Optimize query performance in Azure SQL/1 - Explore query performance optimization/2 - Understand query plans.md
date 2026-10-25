## 🔑 **1. How SQL Server Chooses a Query Plan**

### ✔️ **Cost-Based Optimizer**

- SQL Server uses a **cost-based query optimizer**, meaning it chooses a query plan based on **which option is expected to run the fastest and use the least resources.**
- It calculates the “cost” of possible query plans based on:

  - **Statistics** about your data (how many rows, data distribution, etc.).
  - **Indexes** that could be used.
  - The **complexity of operations** like joins, filters, sorts, etc.

> SQL Server **doesn’t try every possible plan** (there could be thousands), but it uses smart shortcuts (heuristics) to find the most likely good plans and picks the lowest-cost one.

---

## 🔍 **2. Importance of Statistics**

- SQL Server **relies on statistics** to estimate how many rows match parts of your query.
- If stats are out of date, SQL Server might guess wrong, leading to a poor query plan.
- Example: If the optimizer thinks a query returns 10 million rows but it only returns 100, it might give the query way too much memory, wasting resources.

You can update stats manually using:

```sql
UPDATE STATISTICS <table_name>;
```

---

## ⚙️ **3. How a Query is Processed Internally**

1. **Parsing:** Checks the SQL syntax and builds a "parse tree" of the objects involved.
2. **Binding (Algebrizer):** Validates that the tables/columns exist and checks their data types.
3. **Plan Cache Check:** SQL Server looks in the **plan cache** to see if it already has a plan for this query (based on a hash of the query).
4. **Query Optimization:**

   - If no cached plan exists, SQL Server creates a new execution plan using statistics, indexes, and cost estimation.

5. **Execution:** SQL Server runs the query using either a cached plan or the new one.

---

## 📈 **4. Example Query Explained**

```sql
SELECT orderdate,
       AVG(salesAmount)
FROM FactResellerSales
WHERE ShipDate = '2013-07-07'
GROUP BY orderdate;
```

- SQL Server:

  - Checks if `OrderDate`, `ShipDate`, and `SalesAmount` exist.
  - Looks for a cached plan for this exact query.
  - If not found, examines **indexes and stats on `ShipDate`**.
  - If a useful index exists on `ShipDate`, it might use that instead of scanning the whole table.
  - Picks the **lowest-cost plan** and runs the query.

---

## 🧠 **5. Execution Plan Components**

- **Relational Operators:** Steps like index seek, table scan, join, sort, etc.
- **Estimated Row Count:** How many rows SQL Server thinks it will process at each step.
- **Memory Grant:** How much memory SQL Server reserves for the query. If the estimate is wrong, it may:

  - Wait for memory (**RESOURCE_SEMAPHORE wait**), delaying execution.
  - Spill data to **tempdb**, causing slower performance.

---

## 🔬 **6. Types of Query Plans**

| Type                         | Description                                                                  | Usage                                                                       |
| ---------------------------- | ---------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **Estimated Execution Plan** | Shows the plan **before execution**, based on stats.                         | Use to see what SQL _thinks_ it will do. (`SET SHOWPLAN_ALL ON`)            |
| **Actual Execution Plan**    | Shows the plan **after execution**, includes actual row counts and warnings. | Use to compare estimated vs. actual behavior. (`SET STATISTICS PROFILE ON`) |
| **Live Query Statistics**    | Shows a **real-time, animated execution plan** while the query runs.         | Great for troubleshooting long-running queries.                             |

---

## 🛠️ **7. Viewing Plans**

- Most use **SQL Server Management Studio (SSMS)** to see graphical plans.
- You can also view text-based plans using **T-SQL SET commands**.
- Third-party tools like SentryOne and ApexSQL also support plan viewing.

---

### ✅ **Summary**

- SQL Server builds query plans based on **cost estimates from statistics**.
- Plans may be **reused from the plan cache**, or built new.
- Keeping **statistics updated** and **using indexes smartly** improves plan selection.
- Learn to read **estimated vs actual execution plans** to troubleshoot query performance.

---
