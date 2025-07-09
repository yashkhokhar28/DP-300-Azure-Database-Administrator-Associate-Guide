## 🔒 **What Are Locking and Blocking?**

### **Locking**

- SQL Server uses **locks** to protect data and ensure that multiple users don’t change the same data at the same time.
- Locks help maintain **ACID** properties (**Atomicity, Consistency, Isolation, Durability**).
- SQL Server **automatically locks** rows, pages, or tables when reading/writing data.

---

### **Blocking**

- Blocking happens when:

  - **Session A** locks a resource (like a row),
  - and **Session B** tries to access the same resource but must **wait** because the lock is still held by Session A.

- Blocking is **normal** and temporary, but long blocks hurt performance.

---

## 🔍 **Types of Locks in SQL Server**

- Row-level locks: smallest scope → best concurrency.
- Page-level locks: locks a group of rows.
- Table-level locks: entire table → reduces concurrency.
- Lock escalation: if too many row locks (>5000), SQL upgrades to a table lock to save memory.

---

## ⚠️ **When Blocking Becomes a Problem**

1. A process holds a lock for **too long**, causing others to wait.
2. A process **never releases a lock** (for example, if a transaction is left open).
3. **Deadlocks:**

   - Two sessions each hold a lock that the other wants.
   - SQL Server detects this and **kills one query** (called the **deadlock victim**) to resolve it.

---

## 🔁 **Transactions and Locking**

If you forget to end a transaction with **COMMIT** or **ROLLBACK**, the lock stays active.

Example of correct transaction usage:

```sql
BEGIN TRANSACTION

INSERT INTO DemoTable (A) VALUES (1);

COMMIT TRANSACTION
```

If you forget `COMMIT TRANSACTION`, the row remains locked.

---

## 💡 **Row Versioning**

- Instead of locking rows for reads, SQL Server can use **row versioning**.
- Old versions of rows are stored in **TempDB** so that reads and writes don’t block each other.
- Used in **Read Committed Snapshot** and **Snapshot Isolation Levels.**

---

## 🔍 **How to See Open Transactions**

You can run a query to list all running transactions, how long they've been open, and which session is holding them. Example query (already provided in your training) uses dynamic management views (DMVs) like `sys.dm_tran_active_transactions`.

---

## 📊 **Isolation Levels**

Isolation levels control how much of your data is protected during transactions. Higher levels mean better data accuracy but less concurrency.

| Isolation Level          | Allows Dirty Reads? | Locks Held Until End? | Uses Row Versioning?           |
| ------------------------ | ------------------- | --------------------- | ------------------------------ |
| Read Uncommitted         | Yes                 | No                    | No                             |
| Read Committed (default) | No                  | No                    | Optional (snapshot versioning) |
| Repeatable Read          | No                  | Yes (for read rows)   | No                             |
| Serializable             | No                  | Yes (for whole range) | No                             |
| Read Committed Snapshot  | No                  | No                    | Yes                            |
| Snapshot                 | No                  | Yes (versioned)       | Yes                            |

Set it in a session like this:

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

---

## 🛡️ **How to Monitor for Blocking**

- Use **DMVs** like `sys.dm_tran_locks`, `sys.dm_exec_requests` to see blocking sessions.
- Use **Extended Events** for continuous monitoring.
- Look for:

  - Long-running transactions
  - Transactions missing COMMIT
  - Large updates without indexes causing many locks

---

## 🔑 **Key Takeaways**

- Locking is essential but excessive blocking is bad.
- Always close your transactions.
- Use row versioning isolation levels for better read/write concurrency.
- Monitor blocking and deadlocks in production.

---
