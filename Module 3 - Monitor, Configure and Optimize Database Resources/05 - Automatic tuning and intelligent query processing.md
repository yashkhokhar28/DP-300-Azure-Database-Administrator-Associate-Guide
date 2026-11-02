# Automatic tuning and intelligent query processing

> **Exam skills:** Configure database automatic tuning, Identify use cases for intelligent query processing (IQP) features
> **Reading time:** ~9 min

## The big idea
Two features let the database fix some performance problems by itself.
- **Automatic tuning** watches queries and **acts**: it forces a good old plan back, or (in Azure SQL Database) creates and drops indexes.
- **Intelligent Query Processing (IQP)** is a set of smarter ways to run queries. Most of it turns on just by using a new enough **compatibility level**.

Think of a car. Automatic tuning is the mechanic who notices the engine got worse after a part change and puts the old part back. IQP is a newer engine that adjusts itself while you drive.

## Automatic tuning
Automatic tuning uses data from **Query Store** (the query history recorder, lesson 7). Query Store must be **ON** and in **READ_WRITE** mode.

### 1. Automatic plan correction – FORCE_LAST_GOOD_PLAN
- A **plan regression** happens when a new execution plan is slower than an older one. Causes: schema changes, index changes, new statistics, data changes.
- With **FORCE_LAST_GOOD_PLAN**, SQL Server notices the regression and **forces the last known good plan**.
- It then checks the result. If the forced plan is better, it stays. If not, it is **unforced** and a new plan is compiled.

```sql
ALTER DATABASE CURRENT SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON);
SELECT * FROM sys.dm_db_tuning_recommendations;   -- see recommendations
```

### 2. Automatic index management – CREATE_INDEX and DROP_INDEX
- **Azure SQL Database only.**
- **CREATE_INDEX** adds indexes that would help your workload. It waits if the database is busy.
- **DROP_INDEX** drops **unused** (90 days) and **duplicate** indexes. It never drops unique or primary key indexes. On Premium / Business Critical it drops only duplicates.
- If a change makes things worse, Azure **reverts it** automatically.
- Tuning history is kept for **21 days**.

### Where it works
| Platform | FORCE_LAST_GOOD_PLAN | CREATE / DROP INDEX |
|----------|----------------------|---------------------|
| SQL Server 2017+ (VM or on-premises) | Yes | No |
| Azure SQL Database | Yes | **Yes** |
| Azure SQL Managed Instance | Yes | No |

### Defaults and how to configure
- **Azure SQL Database defaults:** FORCE_LAST_GOOD_PLAN = **ON**, CREATE_INDEX = **OFF**, DROP_INDEX = **OFF**.
- Best practice: set options on the **logical server** and let databases **inherit**.
- You can set them in the portal (database or server → **Automatic tuning**) or with T-SQL:

```sql
ALTER DATABASE CURRENT SET AUTOMATIC_TUNING = INHERIT;
ALTER DATABASE CURRENT SET AUTOMATIC_TUNING (CREATE_INDEX = ON, DROP_INDEX = ON);
SELECT name, desired_state_desc, actual_state_desc, reason_desc
FROM sys.database_automatic_tuning_options;
```

**Example:** After a Tuesday deployment, a shop's "find product" query goes from 1 second to 12 seconds because of a new plan. With FORCE_LAST_GOOD_PLAN on, Azure SQL Database spots the regression, forces the old plan, and the query is back to 1 second before customers complain.

**Example 2:** A small SaaS team has no full-time DBA. They turn on **CREATE_INDEX** and **DROP_INDEX** at the server level. Azure adds a missing index on `Orders(CustomerID)` and removes a duplicate index nobody used.

## Intelligent Query Processing (IQP)
IQP is a family of features that fix common slow patterns **without changing your code**. You mostly turn them on by raising the **compatibility level**.

```sql
ALTER DATABASE SalesDb SET COMPATIBILITY_LEVEL = 160;
```
The Learn module says to use **160** on SQL Server 2022 / Azure SQL for the full set. New Azure SQL databases default to **170**.

### The main features and when they help
| Feature | Problem it solves | Compat level |
|---------|-------------------|--------------|
| **Adaptive joins** | Wrong join type because the row estimate was wrong. Picks **hash join** or **nested loops** at run time. Batch mode only. | 140 |
| **Interleaved execution** | Multi-statement table-valued functions (MSTVFs) were guessed at a fixed row count. Now SQL Server runs the function first and uses the real count. | 140 |
| **Memory grant feedback** | Query got too much memory (waste) or too little (spills to tempdb). Next runs get a corrected amount. | 140 batch mode, 150 row mode, 160 percentile + saved in Query Store |
| **Table variable deferred compilation** | Table variables were always guessed as 1 row. Now the real count is used. | 150 |
| **Batch mode on rowstore** | Big aggregations, sorts and joins on normal tables were processed row by row. Now in batches. | 150 |
| **Scalar UDF inlining** | Scalar user-defined functions ran once per row and blocked parallelism. Now they are folded into the main query. | 150 |
| **Approximate count distinct** | `COUNT(DISTINCT)` is slow on huge tables. `APPROX_COUNT_DISTINCT()` is fast, about 2% error with 97% confidence. | Any (function) |
| **Parameter Sensitive Plan (PSP) optimization** | One cached plan bad for skewed data. Keeps **several plans** for one parameterized query. | 160 |
| **DOP feedback** | Parallel queries using more cores than useful. Lowers DOP for repeating queries. Needs Query Store. | 160 |
| **CE feedback** | Wrong estimate model assumptions for repeating queries. Needs Query Store. | 160 |
| **Optimized plan forcing** | High compile cost of Query Store forced plans | Any (SQL Server 2022+, Azure SQL) |
| **Approximate percentile** | Slow exact percentiles: `APPROX_PERCENTILE_CONT/DISC` | 110+ (SQL Server 2022+, Azure SQL) |
| **Optional parameter plan optimization (OPPO)** | Bad plans for `@p IS NULL OR col = @p` patterns | 170 |
| **CE feedback for expressions** | Learns estimate choices for repeating expressions | 170 (Azure SQL DB) / 160 (SQL Server 2025) |
| **OPTIMIZED_SP_EXECUTESQL** | Compile storms from `sp_executesql` | Database-scoped config (SQL Server 2025, Azure SQL DB) |

- **DOP feedback** is **off by default in SQL Server 2022**: `ALTER DATABASE SCOPED CONFIGURATION SET DOP_FEEDBACK = ON;`
- On **Managed Instance**, DOP feedback, OPPO, CE feedback for expressions and optimized plan forcing need the **SQL Server 2025** or **Always-up-to-date** update policy.

### Use-case examples
- **Memory grant feedback:** A nightly query keeps spilling to tempdb because it asked for too little memory. After the first run, the grant is raised and the spills stop.
- **Scalar UDF inlining:** A shop calls `dbo.GetTax(price)` on every row of a 5-million-row table. After moving to compat 150+, the function is inlined and the query can run in parallel.
- **PSP optimization:** `GetOrders(@CustomerID)` is fast for small customers and slow for one huge customer. PSP keeps separate plans for small and big parameter values.

## Remember for the exam
- **FORCE_LAST_GOOD_PLAN** = automatic plan correction; works on SQL Server 2017+, SQL DB and MI.
- **CREATE_INDEX / DROP_INDEX** = **Azure SQL Database only**.
- Azure SQL DB defaults: plan correction **ON**, index create/drop **OFF**.
- Configure at the **server** and let databases **INHERIT**.
- Automatic tuning needs **Query Store ON, READ_WRITE**.
- DMVs: `sys.dm_db_tuning_recommendations` (suggestions), `sys.database_automatic_tuning_options` (settings).
- Most IQP features turn on by **compatibility level**: 140, 150, 160, 170.
- Table variable guessed as 1 row → **table variable deferred compilation**. MSTVF bad estimate → **interleaved execution**. Tempdb spills → **memory grant feedback**. Slow scalar function → **scalar UDF inlining**.
- Skewed parameter values → **PSP optimization** (160).

## Quick check
1. You want Azure SQL Managed Instance to create missing indexes automatically. Is this possible?
   <details><summary>Answer</summary>No. Automatic index management (CREATE_INDEX/DROP_INDEX) is Azure SQL Database only. MI supports FORCE_LAST_GOOD_PLAN.</details>
2. A query that uses a table variable with 100,000 rows gets a bad plan because SQL Server guesses 1 row. Which IQP feature helps, and how do you enable it?
   <details><summary>Answer</summary>Table variable deferred compilation. Set the compatibility level to 150 or higher.</details>
3. A query keeps spilling to tempdb because its memory grant is too small. Which IQP feature fixes this over time?
   <details><summary>Answer</summary>Memory grant feedback. It adjusts the memory grant on later runs.</details>
