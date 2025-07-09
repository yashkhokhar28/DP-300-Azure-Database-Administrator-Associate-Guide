## ✅ **Why Index Tuning Matters**

Indexes improve **read/query performance** by allowing SQL Server to **quickly find rows**, instead of scanning the whole table.
But:

- Too many indexes = bad for **write performance** (INSERT/UPDATE/DELETE need to maintain the indexes too).
- Goal: **Balance between reads and writes.**

---

## 🔍 **Steps to Tune Indexes**

### 1. **Evaluate Existing Index Usage**

Use these system views:

- `sys.dm_db_index_operational_stats` → shows index usage patterns
- `sys.dm_db_index_usage_stats` → shows how often indexes are read, updated, or scanned.

Example query:

```sql
SELECT OBJECT_NAME(i.object_id), i.name, u.user_seeks, u.user_scans, u.user_lookups, u.user_updates
FROM sys.indexes AS i
JOIN sys.dm_db_index_usage_stats AS u
ON i.object_id = u.object_id AND i.index_id = u.index_id
WHERE OBJECTPROPERTY(i.object_id,'IsUserTable') = 1;
```

If an index has **very low seeks/scans but high updates**, it might be **useless or costly for writes.**

---

### 2. **Remove Duplicate/Unused Indexes Carefully**

⚠️ Don't drop an index just because it's rarely used. It might be critical during:

- Month-end reports
- Yearly audits

**Pro Tip:** Create such indexes temporarily before the report runs.

---

### 3. **Tune Expensive Queries**

Use **Query Store** or **Extended Events** to find slow queries. Then:

- Add new indexes.
- Test them in a **non-production environment first.**
- Compare query plans **before & after** adding the index.

---

### 4. **Optimize Column Order in Indexes**

- First column = most important.
- Put **WHERE clause columns first.**
- Example: If your query is

```sql
SELECT * FROM Orders WHERE CustomerID = 10 AND OrderDate > '2025-01-01'
```

Your index should be:

```sql
CREATE INDEX IX_Orders_CustomerID_OrderDate ON Orders (CustomerID, OrderDate);
```

Put **equality filter (`=`)** columns before **range filters (`>`, `<`, `BETWEEN`).**

---

### 5. **Minimize the Number of Indexes**

Having **the smallest number of indexes** reduces storage and maintenance overhead.

- Add columns to existing indexes where useful (**included columns**).
- Example: Avoid creating 3 separate indexes if 1 covering index works.

---

## ⏸️ **Resumable Index Operations**

### Problem:

If you create or rebuild a large index in production, it can:

- Take a long time
- Block users
- Fail halfway due to server restarts

### Solution: **Resumable Indexes (SQL Server 2019+)**

- You can **pause** and **resume** index operations.
- Useful during **maintenance windows.**

Example to create a resumable online index:

```sql
CREATE INDEX IX_Customer_PersonID_ModifiedDate
    ON Sales.Customer (PersonID, StoreID, TerritoryID, AccountNumber, ModifiedDate)
WITH (RESUMABLE = ON, ONLINE = ON);
```

Pause it midway:

```sql
ALTER INDEX IX_Customer_PersonID_ModifiedDate ON Sales.Customer PAUSE;
```

Check status:

```sql
SELECT * FROM sys.index_resumable_operations;
```

👉 **Note:** Resumable is only available for **online index operations.**

---

## 🎯 **Best Practices Summary**

| Tip                                          | Why it matters        |
| -------------------------------------------- | --------------------- |
| Remove unused indexes (carefully)            | Reduces overhead      |
| Keep leading index columns selective         | Optimizes query plans |
| Avoid too many indexes on write-heavy tables | Reduces write costs   |
| Test in dev/test before production           | Avoids surprises      |
| Use resumable indexes for big tables         | Minimizes downtime    |

---
