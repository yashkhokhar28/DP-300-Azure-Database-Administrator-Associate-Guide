### ✅ **Azure SQL: Query Performance Insight — Explained**

---

### 📊 **What Is Query Performance Insight (QPI)?**

**Query Performance Insight (QPI)** is a **built-in Azure SQL Database tool** that helps you:

- Identify **resource-heavy queries**
- Spot **long-running queries**
- Filter and analyze **performance metrics** like **CPU**, **Data I/O**, and **Log I/O**

It simplifies what used to require complex SQL scripts or SQL Profiler traces in on-premises SQL Server.

---

### 🔍 **Key Benefits**

| **Feature**                      | **Purpose**                                                                          |
| -------------------------------- | ------------------------------------------------------------------------------------ |
| **No manual tracing needed**     | Automatically gathers query performance data.                                        |
| **Easy to use**                  | Accessible from the Azure portal UI.                                                 |
| **Breakdown by resource**        | See which queries use the most **CPU**, **disk I/O**, or **log I/O**.                |
| **Integration with Query Store** | Click a query to find its details in the **Query Store** for deeper troubleshooting. |
| **Built-in visualizations**      | View trends over time and quickly spot spikes.                                       |

---

### 🛠️ **How to Access Query Performance Insight**

1. Go to your **Azure SQL Database** in the **Azure portal**.
2. Under the **Intelligent Performance** section, click on **Query Performance Insight**.
3. You'll see **3 main analysis views:**

   - ➤ **Resource Consuming Queries**
   - ➤ **Long Running Queries**
   - ➤ **Custom Filter**

---

### 🔬 **Modes of Analysis**

| Mode                           | Description                                                                                   |
| ------------------------------ | --------------------------------------------------------------------------------------------- |
| **Resource Consuming Queries** | Shows top 5 queries sorted by CPU, Data I/O, or Log I/O.                                      |
| **Long Running Queries**       | Displays top 5 queries that have taken the longest execution times in the past 24 hours.      |
| **Custom Filter**              | Lets you build custom views: choose the metric (**CPU**, **I/O**, **memory**) and time frame. |

---

### 🎯 **Example Use Case: Finding Top CPU Queries**

- Select **Resource Consuming Queries**.
- Sort by **CPU usage**.
- See a graph and list of the top 5 queries consuming the most CPU.
- Click on a query to view its **Query ID** and **T-SQL text**.
- Use this ID to find its **execution plan** in the Query Store (for deeper tuning).

---

### 🔍 **Key Metrics Available**

| Metric       | Meaning                                                 |
| ------------ | ------------------------------------------------------- |
| **CPU**      | Processor time used by the query.                       |
| **Data I/O** | Data read/written by the query.                         |
| **Log I/O**  | Transaction log activity caused by the query.           |
| **Memory**   | (In Custom) Shows queries with high memory consumption. |

---

### 🔗 **Query Store Integration**

- Each query in QPI has a **Query ID**, which matches the ID in the **Query Store**.
- The Query Store provides **execution plans**, runtime stats, and historical performance.
- This allows you to go from **“what’s slow”** to **“why is it slow”**.

---

### ⚠️ **Limitations**

- QPI **doesn’t show execution plans directly**, but you can extract them from the Query Store.
- Shows only **aggregated data**, not individual query executions.

---

### ✔️ **Summary Takeaways**

- **Query Performance Insight helps identify bad queries in Azure SQL easily.**
- It’s **visual, fast, and integrated into the Azure portal**.
- Use it to pinpoint expensive queries by **CPU, I/O, or duration**, then use the Query Store for deeper troubleshooting.

---
