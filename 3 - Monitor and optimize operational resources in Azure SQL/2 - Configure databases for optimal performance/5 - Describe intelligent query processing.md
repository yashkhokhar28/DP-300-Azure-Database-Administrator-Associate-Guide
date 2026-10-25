## 🌟 **What is Intelligent Query Processing (IQP)?**

Intelligent Query Processing is a group of **automatic optimizations** built into SQL Server and Azure SQL. These optimizations fix common performance issues (anti-patterns) and improve execution plans, making workloads run faster **with minimal changes required from developers**.

IQP features are tied to compatibility levels **140 / 150 / 160 / 170**; the Learn module now says use **160** on SQL Server 2022 / Azure SQL for the full set (new Azure SQL databases default to **170**): 🔄

```sql
ALTER DATABASE [WideWorldImportersDW] SET COMPATIBILITY_LEVEL = 160; -- 🔄 (was 150)
```

---

## ⚙️ **Main IQP Feature Families**

---

### 1. 🔧 **Adaptive Query Processing**

These features allow SQL Server to **adjust query execution plans dynamically**, depending on what happens at runtime.

- **Adaptive Joins**: SQL Server waits until it knows how many rows a join will process, then chooses the most efficient join type (**Hash Join** or **Nested Loops**).
  ➤ Example: If few rows are returned, it uses Nested Loops; otherwise, it uses Hash Join.
  ➤ Works only in **batch mode**.

- **Interleaved Execution (for MSTVFs)**:

  - MSTVFs (**Multi-Statement Table-Valued Functions**) used to assume they returned 1 or 100 rows, leading to poor plans.
  - Now, SQL Server **runs the MSTVF first to get the actual row count**, and then optimizes the remaining query based on that number.

- **Memory Grant Feedback**:

  - SQL Server estimates how much memory a query needs.
  - If it guesses wrong (too much or too little), the query might fail or be inefficient.
  - Memory Grant Feedback adjusts the memory for future executions to avoid spills to **tempdb** or wasting memory. Batch mode at compat **140**, row mode at **150**; percentile + persistence (in Query Store) at **160**. 🔄

---

### 2. 📈 **Table Variable Deferred Compilation**

Previously, **table variables** always assumed they had **only one row**, which often gave SQL Server a bad estimate for how to execute queries.
Now, SQL Server **waits until the query runs to count the actual rows** before compiling the query plan, giving more accurate plans.

---

### 3. ⚙️ **Batch Mode on Rowstore**

Batch mode was originally only for **columnstore indexes**, but now **rowstore tables** (regular tables) can also process rows in batches.
This improves CPU efficiency, especially for queries with:

- Aggregations
- Sorts
- Joins

Batch processing reduces overhead by handling **multiple rows at a time** instead of one by one.

---

### 4. 🔍 **Scalar UDF Inlining**

Scalar **user-defined functions (UDFs)** used to run slowly because:

- They were executed **once per row** (very inefficient).
- They couldn't participate in parallelism.
- Their cost wasn’t properly estimated in query plans.

Now, SQL Server **inlines scalar UDFs directly into the query plan** as subqueries, making them faster and parallelizable.

---

### 5. 🔢 **Approximate Count Distinct**

Doing a `COUNT(DISTINCT column)` on large tables is slow.

The new `APPROX_COUNT_DISTINCT()` function gives you a **fast estimate** with:

- \~2% error margin
- 97% confidence
- Much faster results for large datasets (great for analytics)

---

### 6. 🆕 **Newer IQP features (compat 160 / 170)** 🔄

| Feature | Compat level | Notes 🔄 |
| --- | --- | --- |
| **Parameter Sensitive Plan (PSP) optimization** | 160 | Several cached plans for one parameterized query (skewed data). 🔄 |
| **DOP feedback** | 160 | Lowers DOP for repeating, inefficient parallel queries; needs Query Store; **off by default in SQL Server 2022** → `ALTER DATABASE SCOPED CONFIGURATION SET DOP_FEEDBACK = ON`. 🔄 |
| **CE feedback** | 160 | Adjusts cardinality-estimation model assumptions for repeating queries; needs Query Store. 🔄 |
| **Optimized plan forcing** | any (SQL Server 2022+, Azure SQL) | Reduces compile overhead for Query Store–forced plans. 🔄 |
| **Approximate percentile** (`APPROX_PERCENTILE_CONT/DISC`) | 110+ | SQL Server 2022+, Azure SQL. 🔄 |
| **Optional parameter plan optimization (OPPO)** | 170 | Picks the plan at runtime for `@p IS NULL OR col = @p` patterns. 🔄 |
| **CE feedback for expressions** | 170 (Azure SQL DB) / 160 (SQL Server 2025) | Learns CE model choices for repeating expressions. 🔄 |
| **OPTIMIZED_SP_EXECUTESQL** | database scoped config (SQL Server 2025, Azure SQL DB) | Reduces compilation storms from `sp_executesql`. 🔄 |

- On **Managed Instance**, DOP feedback, OPPO, CE feedback for expressions and optimized plan forcing need the **SQL Server 2025** or **Always-up-to-date** update policy. 🔄

---

## ✅ **Summary Table**

| Feature                             | Problem Solved                                | Benefit                              |
| ----------------------------------- | --------------------------------------------- | ------------------------------------ |
| Adaptive Joins                      | Wrong join type due to bad row estimates      | Picks join type at runtime           |
| Interleaved Execution               | Bad estimates for MSTVFs                      | More accurate plans                  |
| Memory Grant Feedback               | Wrong memory allocation                       | Reduces tempdb spills & memory waste |
| Table Variable Deferred Compilation | Wrong row count estimates for table variables | Better execution plans               |
| Batch Mode on Rowstore              | Row-by-row processing overhead                | Faster CPU processing                |
| Scalar UDF Inlining                 | Slow, iterative scalar function execution     | Parallel, faster queries             |
| Approximate Count Distinct          | Slow distinct count on big tables             | Fast, approximate results            |

---

## 🔑 **Key Takeaways**

- Most IQP features are **automatic** once your database is on the right compatibility level (140/150; **160** for PSP and DOP/CE feedback; **170** for OPPO). 🔄
- No need to rewrite queries or indexes.
- They solve long-standing performance problems with common T-SQL patterns.
- Improves query processing flexibility, accuracy, and speed.

---
