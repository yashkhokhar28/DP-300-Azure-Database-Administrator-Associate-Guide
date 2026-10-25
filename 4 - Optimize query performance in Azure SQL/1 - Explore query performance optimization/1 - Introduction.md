### 🔍 **1. Query Execution Plans**

- **What they are:** Execution plans show how SQL Server processes a query.
- **Why they're important:** By analyzing these plans, you can:

  - See which parts of a query are slow.
  - Identify inefficient operations (e.g., full table scans instead of index seeks).
  - Make adjustments to improve speed and reduce resource usage.

---

### 📊 **2. Query Store**

- **What it does:** Query Store keeps a history of executed queries and their performance.
- **Why it's helpful:** You can:

  - Find your **slowest or most expensive queries**.
  - Track how a query’s performance changes over time.
  - Analyze which query plans work best and which cause problems.

---

### 🔒 **3. Locking and Blocking**

- **Why they matter:** In databases, multiple users might try to access or modify the same data at the same time.
- **What SQL Server does:** It uses locks to prevent **conflicts** and maintain **data consistency**, but too much locking can cause **blocking**, slowing things down.

---

### ⚙️ **4. Isolation Levels**

- **What they are:** Isolation levels control how much one transaction sees changes from another transaction before they are committed.
- **Why adjust them:** Lower isolation levels improve speed but risk inconsistent reads. Higher levels protect data integrity but may cause more blocking.
- You need to **balance data consistency and performance**, depending on your workload.

---

### ✅ **Overall Summary**

- Learn to read **execution plans**.
- Use **Query Store** to monitor and optimize queries.
- Understand and tune **locking**, **blocking**, and **isolation levels** to handle concurrency efficiently.

---
