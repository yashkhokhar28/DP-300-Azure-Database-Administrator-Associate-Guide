## ✅ **Goal of Query Troubleshooting**

When a query runs slowly, your main job is to:

1. **Find the query causing performance issues.**
2. **Look at its execution plan** to see why it’s slow.

There are two common types of slow queries:

- **Always slow:** bad query design, missing indexes, bad plans, etc.
- **Sometimes slow:** the same query runs fast sometimes and slow other times. Usually caused by:

  - Blocking (waiting for locks).
  - Data differences (e.g., one query handles 10 rows, another handles 1 million rows).
  - Hardware overload.

---

## ⚙️ **Reasons for Bad Performance**

### 1. **Hardware Issues**

- Usually affects many queries, not just one.
- Symptoms:

  - High **CPU usage** → `% Processor Time`, `SOS_SCHEDULER_YIELD`, `CXPACKET` waits.
  - Slow disk reads/writes → `PAGEIOLATCH_SH` waits.

- Tools to check: Windows Performance Monitor, SQL Server wait types.

### 2. **Bad Query Patterns**

#### a. **Row-based processing (Bad)**

- Cursors or loops (WHILE) process one row at a time → slow.
- **Set-based operations** (SELECT, UPDATE on whole sets) are better.

#### b. **Table-Valued Functions (TVFs)**

- Multi-statement TVFs are slow because SQL Server **guesses row counts**, leading to poor plans.
- Inline TVFs are better.

#### c. **Scalar Functions**

- Also slow down queries, though SQL 2017+ has some improvements.

### 3. **Non-SARGable Queries**

A query is **SARGable** (Search ARGument Able) if SQL can efficiently use an index.

Bad Example (non-SARGable):

```sql
WHERE LEFT(City, 1) = 'M'
```

Good Example (SARGable):

```sql
WHERE City LIKE 'M%'
```

If your predicate uses a function (`LEFT`, `CONVERT`, etc.), SQL can’t do an **index seek** and has to scan.

---

### 4. **Missing or Poorly Designed Indexes**

- Most common cause of slow queries.
- Fix: add the right index to support the WHERE clauses and JOINs.
- SQL Server suggests missing indexes in the execution plan and in `sys.dm_db_missing_index_details`.

### 5. **Outdated Statistics**

- SQL uses **statistics** to guess how many rows a query will return.
- Old SQL Server versions didn’t update stats frequently enough.
- Newer versions update them more dynamically.
- You can check stats with `sys.dm_db_stats_properties`.

---

### 6. **Bad Optimizer Choices**

Sometimes the SQL optimizer makes a wrong guess.

- Fixes:

  - Query hints (`OPTION (RECOMPILE)`, `OPTIMIZE FOR`).
  - Forced execution plans.
  - Contact Microsoft if really weird.

Example hint:

```sql
OPTION (OPTIMIZE FOR (@city_name = 'Seattle'))
```

---

## 🔍 **Special Issue: Parameter Sniffing**

When SQL Server builds an execution plan, it uses the **first value of a parameter** it sees.
If that value is unusual, the plan may be bad for other parameter values.

Example:

```sql
EXEC GetAccountID 42;
```

If `42` matches millions of rows, but later you run it with `5` (which matches 10 rows), the original plan may be wrong.

### Solutions:

| Method                 | Trade-Off                                          |
| ---------------------- | -------------------------------------------------- |
| `OPTION (RECOMPILE)`   | Always compiles fresh plan, uses more CPU.         |
| `OPTIMIZE FOR UNKNOWN` | Ignores parameter and uses average stats.          |
| Conditional RECOMPILE  | Adds complexity but optimizes for bad values only. |

Example of a conditional RECOMPILE:

```sql
IF @SalesPersonID IS NULL
BEGIN
   -- Add RECOMPILE hint
END
```

---

## 🛡️ **Summary of Problems and Fixes**

| Problem                    | Fix                                               |
| -------------------------- | ------------------------------------------------- |
| CPU or Disk slow           | Check hardware and waits.                         |
| Poor query design          | Use set-based queries, avoid bad functions.       |
| Non-SARGable WHERE clauses | Rewrite queries to use index-friendly predicates. |
| Missing indexes            | Add useful indexes, verify usage.                 |
| Outdated stats             | Update stats or tune auto-update.                 |
| Optimizer mistakes         | Use hints or force better plans.                  |
| Parameter sniffing         | Recompile or use OPTIMIZE FOR UNKNOWN.            |

---
