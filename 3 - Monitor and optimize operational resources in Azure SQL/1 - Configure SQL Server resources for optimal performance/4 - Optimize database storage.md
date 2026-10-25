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
- **Azure SQL Database** has only **one data file** (Hyperscale typically several) — you can't add files. 🔄

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
- **Azure SQL Database**: the number of tempdb files scales with vCores (e.g., **2 vCores = 4 files**), **max 16**; not configurable via T-SQL. 🔄
- **SQL Server**: setup creates one file per logical processor **up to 8**; size them evenly; if contention persists add files in **multiples of 4** (up to the number of cores). 🔄
- **Managed Instance**: **12 tempdb data files** by default, and you **can** change the number (max 128), growth increment and max size — settings persist across restarts/failovers. 🔄
- Memory-optimized tempdb metadata (SQL Server 2019+) is **not available** in Azure SQL Database or Managed Instance. 🔄

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
