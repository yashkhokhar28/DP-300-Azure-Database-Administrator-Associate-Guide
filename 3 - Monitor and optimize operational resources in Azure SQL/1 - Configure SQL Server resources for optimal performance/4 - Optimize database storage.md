## ✅ **How to Optimize Database Storage in Azure SQL and SQL Server**

---

### ⚙️ **Key Factors Affecting Database Storage Performance**

1. **I/O Performance**

   - **IOPS (Input/Output Operations Per Second):**
     Measures how many read/write operations your storage can perform. If your database is slow during queries, it may lack enough IOPS.
   - **I/O Latency:**
     The time it takes to complete each I/O operation. Lower latency = faster response.
   - **Azure Recommendations:**

     - Use **Business Critical** or **Hyperscale** tiers for better IOPS and low latency.
     - On **Managed Instance**, you can also adjust file sizes and the number of files for better performance.

---

### 📂 **Files & Filegroups (SQL Server concept)**

- In **on-prem SQL Server**, DBAs split data across multiple physical disks using **files and filegroups** to improve I/O.
- In **Azure SQL Database**, you don’t control physical disks, but Azure automatically optimizes storage to meet your performance tier.
- **Managed Instance** allows you to create multiple files (but still abstracts physical placement).

---

### ⚖️ **Proportional Fill Concept**

When SQL Server writes data across multiple files:

- It **fills files proportionally** based on their size.

Example:

| File  | Size  | % of Data Written |
| ----- | ----- | ----------------- |
| File1 | 10 GB | \~90%             |
| File2 | 1 GB  | \~10%             |

⚠️ In systems like **tempdb**, this uneven filling can cause bottlenecks if files are unbalanced.

---

### 🛡️ **Tempdb Configuration**

Tempdb is a special system database used for:

- Temporary tables
- Sorting operations
- Intermediate query results
- Row versioning

Key recommendations:

- Tempdb is stored on fast local SSDs in Azure.
- SQL Server creates multiple tempdb files (by default, one per vCore, max 16 for Azure SQL DB).
- In SQL Server, you should size tempdb files evenly and increase the number if needed (up to number of CPU cores).

Tempdb settings:

| Setting                        | Value                     |
| ------------------------------ | ------------------------- |
| MIXED_PAGE_ALLOCATION          | OFF (Recommended default) |
| AUTOGROW_ALL_FILES             | ON                        |
| Tempdb file placement on Azure | Local SSD                 |

---

### 🛠️ **Database Configuration (ALTER DATABASE)**

You can configure various SQL settings to optimize performance. Some important ones:

- Recovery model: Azure SQL DB uses **Full Recovery Model** for reliability (except for tempdb, where minimal logging is allowed).
- Use **ALTER DATABASE** and **ALTER DATABASE SCOPED CONFIGURATION** for changes.

---

### 🧠 **MAXDOP (Max Degree of Parallelism)**

Controls how many CPU cores a query can use:

- **High MAXDOP:** More cores → faster query → but higher memory consumption
- **Low MAXDOP:** Fewer cores → less memory pressure → more stable performance on busy systems

How to set it:

- Azure SQL: Use `ALTER DATABASE SCOPED CONFIGURATION`
- Managed Instance: Can also use `sp_configure` and Resource Governor
- Can override with query hints.

---

### 🔑 **Summary of Best Practices**

✔ Choose the right Azure service tier for your IOPS and latency needs.
✔ Balance file sizes (Proportional Fill).
✔ Optimize **tempdb** by scaling files based on vCores.
✔ Configure **MAXDOP** to balance parallelism and memory use.
✔ Use Azure’s built-in optimizations to manage storage performance without worrying about physical disks.

---
