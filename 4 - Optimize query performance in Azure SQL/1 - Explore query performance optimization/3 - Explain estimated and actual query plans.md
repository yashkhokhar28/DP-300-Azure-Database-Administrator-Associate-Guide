## 🔍 **1. What’s the Difference Between Estimated and Actual Plans?**

| **Estimated Execution Plan**                      | **Actual Execution Plan**                               |
| ------------------------------------------------- | ------------------------------------------------------- |
| Shows what SQL Server **thinks will happen**      | Shows **what actually happened** during query execution |
| No need to run the query                          | The query **must run**                                  |
| Contains **row count and cost estimates**         | Contains **real row counts and execution statistics**   |
| Useful when you're designing or analyzing a query | Useful for troubleshooting **performance problems**     |

✅ **Key point:**
The plan structure (the operations used and their order) is almost always the **same**, but only the actual plan has **runtime stats**.

---

## 🛠️ **2. How to Generate Execution Plans in SSMS**

- **Estimated Plan:**

  - Shortcut: **Ctrl + L**
  - Button: “Display Estimated Execution Plan”
  - Does _not_ run the query.

- **Actual Plan:**

  - Shortcut: **Ctrl + M** (then run the query)
  - Button: “Include Actual Execution Plan”
  - The query runs, and you get the real stats.

⚠️ **Tip:** Don’t generate plans frequently on production databases. Even viewing plans creates overhead.

---

## 🧠 **3. Example Query Explained**

```sql
SELECT [stockItemName],
       [UnitPrice] * [QuantityPerOuter] AS CostPerOuterBox,
       [QuantityonHand]
FROM [Warehouse].[StockItems] s
JOIN [Warehouse].[StockItemHoldings] sh ON s.StockItemID = sh.StockItemID
ORDER BY CostPerOuterBox;
```

- Joins two tables on `StockItemID`.
- SQL Server shows you operations like **Clustered Index Scan**, **Join**, and **Sort** in the plan.

---

## 🔧 **4. Reading a Query Plan**

- **Icons:** Each step in the plan is an operator (Scan, Join, Sort, etc.)
- **Cost %:** Shows how expensive each step is relative to the whole query.
- **Arrow Thickness:** Thicker arrows = more rows processed between steps.
- **Right to Left, Top to Bottom:** That’s the order SQL processes the steps.
- **Tooltip:** Hover to see stats like estimated rows.
- **Properties Pane:** Right-click an operator → **Properties** for deeper details.

Example insight:

> If a **Clustered Index Scan** shows up where an **Index Seek** would be better, maybe you need to create a **Non-Clustered Index** on the join column.

---

## ⚙️ **5. Lightweight Query Profiling**

**Problem:**
Old profiling methods had **high overhead** (could slow queries by 75%!).

**Solution:**
Lightweight Profiling has much less overhead (**around 2%**). It can:

- Track rows and I/O per operator.
- Capture the actual execution plan without slowing down your queries too much.

### Ways to enable lightweight profiling:

| Method                                                 | Usage                           |
| ------------------------------------------------------ | ------------------------------- |
| Trace flag 7412                                        | Enables globally                |
| **USE HINT ('QUERY_PLAN_PROFILE')**                    | Enables for **one query**       |
| Database Scoped Config: `LIGHTWEIGHT_QUERY_PROFILING`  | Enables for a specific database |
| Available by default in SQL Server 2019+ and Azure SQL |                                 |

---

## 🛡️ **6. Getting the Last Actual Plan**

Sometimes, you want to see the **actual execution plan from the last run of a query**. You can use:

```sql
SELECT *
FROM sys.dm_exec_cached_plans AS cp
CROSS APPLY sys.dm_exec_sql_text(plan_handle) AS st
CROSS APPLY sys.dm_exec_query_plan_stats(plan_handle) AS qps;
```

✅ This lets you:

- See **the actual number of rows returned**.
- Review plans **without re-running the query**.

---

## ✅ **Summary: When to Use Which Plan?**

| **When writing the query:**                | Use Estimated Plan                                             |
| ------------------------------------------ | -------------------------------------------------------------- |
| **When troubleshooting slow performance:** | Use Actual Plan                                                |
| **On production servers:**                 | Use Lightweight Profiling or cached plans to minimize overhead |

---
