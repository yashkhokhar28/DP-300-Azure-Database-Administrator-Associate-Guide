## 🔍 **What Are Query Hints?**

- **Query hints** are options you add to your query to force the SQL engine to:

  - Use specific operations (like a certain type of join).
  - Recompile the plan every time.
  - Control how much CPU power is used, etc.

- You add them using the `OPTION()` clause at the end of your query.

---

## ✨ **Examples of Common Query Hints**

| Hint           | What It Does                                                    | Example                                  |                              |                      |
| -------------- | --------------------------------------------------------------- | ---------------------------------------- | ---------------------------- | -------------------- |
| `MAXDOP <n>`   | Limits the number of CPU cores used.                            | `OPTION (MAXDOP 2)`                      |                              |                      |
| `RECOMPILE`    | Forces query plan to be rebuilt every time.                     | `OPTION (RECOMPILE)`                     |                              |                      |
| `FAST <n>`     | Returns the first _n_ rows as fast as possible.                 | `OPTION (FAST 10)`                       |                              |                      |
| `OPTIMIZE FOR` | Pretend a variable has a specific value during optimization.    | `OPTION (OPTIMIZE FOR (@ProductID = 5))` |                              |                      |
| \`{LOOP        | MERGE                                                           | HASH} JOIN\`                             | Forces a specific join type. | `OPTION (HASH JOIN)` |
| `USE PLAN`     | Forces the query to use a predefined execution plan (from XML). | `OPTION (USE PLAN N'<xml>...</xml>')`    |                              |                      |

---

## ⚠️ **Why Be Careful with Hints?**

- **Don't use them unless necessary!**
- SQL Server normally chooses the best plan based on:

  - **Indexes**
  - **Statistics**
  - **Data distribution**

- If you use hints permanently:

  - You might block the optimizer from using better features in newer SQL Server versions.
  - Database structure changes (e.g., new indexes) may no longer improve query plans.

---

## 🔧 **Example Queries**

**Example 1: Parallelism limited to 2 cores**

```sql
SELECT ProductID, SUM(LineTotal)
FROM Sales.SalesOrderDetail
WHERE UnitPrice < 5.00
GROUP BY ProductID
OPTION (MAXDOP 2);
```

**Example 2: Recompile every time**

```sql
SELECT City
FROM Person.Address
WHERE StateProvinceID = 15
OPTION (RECOMPILE);
```

**Example 3: Combine multiple hints**

```sql
SELECT ProductID, SUM(LineTotal)
FROM Sales.SalesOrderDetail
WHERE UnitPrice < 5.00
GROUP BY ProductID
OPTION (HASH GROUP, FAST 10);
```

---

## ⚙️ **What Are Query Store Hints?**

In **Azure SQL and SQL Server 2022+**, there is a better way to apply hints **without changing the query text itself** — called **Query Store hints**.

### Example Use Case:

- Your app has a hard-coded query.
- You can't change the app code.
- But you want that query to use `RECOMPILE`.

### Steps:

1. Find the `query_id` of the query in Query Store:

```sql
SELECT q.query_id, qt.query_sql_text
FROM sys.query_store_query_text qt
JOIN sys.query_store_query q ON qt.query_text_id = q.query_text_id
WHERE qt.query_sql_text LIKE N'%ORDER BY CustomerName DESC%';
```

2. Apply a hint to it:

```sql
EXEC sys.sp_query_store_set_hints
    @query_id = 42,
    @query_hints = N'OPTION(RECOMPILE, MAXDOP 1)';
```

---

## 🎯 **When Should You Use Query Hints?**

| When it's OK to use                             | When to avoid                           |
| ----------------------------------------------- | --------------------------------------- |
| Fixing a **critical performance issue quickly** | For long-term/permanent solutions       |
| Application query **can't be changed**          | When query optimizer already works well |
| Optimizing specific, isolated queries           | For most queries in the system          |

---

## ✅ **Key Takeaways**

- Use query hints **sparingly** and **only after testing.**
- Prefer fixing the **root cause** (e.g., missing index, wrong stats).
- Use **Query Store hints** for production systems where query text changes are impossible.

---
