## 🔑 **Key Concepts Explained**

---

### 📋 **What Are Database Scoped Configurations?**

SQL Server offers many **configuration settings** that **affect query performance and behavior**.
Some of these are set using:

- `ALTER DATABASE SCOPED CONFIGURATION`
- `ALTER DATABASE`

👉 Both commands configure settings **at the database level**, but there's no technical difference between them besides syntax and which options they support.

---

### ⚙️ **Examples of Database-Level Options**

| **Option Name**                   | **Purpose**                                                                                      |
| --------------------------------- | ------------------------------------------------------------------------------------------------ |
| **Recovery Model**                | Full or Simple. Controls how much transaction log data is retained for recovery.                 |
| **Automatic Tuning**              | Allows enabling/disabling automatic fixes, like forcing the last known good plan.                |
| **Auto Create/Update Statistics** | Keeps query statistics current. Improves query optimization.                                     |
| **Query Store Options**           | Manages Query Store settings to capture query performance history.                               |
| **Snapshot Isolation**            | Enables Snapshot Isolation (SI) and Read Committed Snapshot Isolation (RCSI), reducing blocking. |

---

### ➕ **Advanced Database Scoped Options**

Some options that used to be server-level can now be **set per database**, giving you more flexibility:

| **Advanced Option**                    | **Purpose/Usage**                                                                                                           |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **MAXDOP (Max Degree of Parallelism)** | Limits how many CPU cores a **single query** can use within this database, regardless of the server setting.                |
| **Legacy Cardinality Estimation**      | Lets older databases use the **older query optimizer row estimation model**, improving performance for some legacy queries. |
| **Last Query Plan Stats**              | Captures the **last actual execution plan** for a query. Helps with troubleshooting.                                        |
| **Optimize for Ad Hoc Workloads**      | Avoids caching full query plans for **single-use queries**, reducing memory pressure.                                       |

---

### 🛡️ **Database Compatibility Level**

- Every database has a **Compatibility Level**, such as 120, 140, or 150.

- This controls how the **query optimizer behaves**, helping preserve compatibility with older SQL Server versions.

  - Example: A database originally built for SQL Server 2014 might use **compatibility level 120**.

- **Newer compatibility levels** (140, 150) enable features like **Intelligent Query Processing (IQP)**, improving query performance without changing the queries themselves.

- You can upgrade compatibility level later, after testing.

---

### ✅ **Why Use Scoped Configuration?**

| **Scenario**                                      | **Scoped Configuration Example**                        |
| ------------------------------------------------- | ------------------------------------------------------- |
| Multi-tenant SQL Server hosting several databases | Give each database its own performance settings.        |
| Modernizing one database without affecting others | Enable IQP on one database only.                        |
| Troubleshooting a single workload                 | Enable legacy cardinality estimation for that database. |

---

### 🔨 **Common Syntax Examples**

```sql
-- Example: Enable legacy cardinality estimation at the database level
ALTER DATABASE SCOPED CONFIGURATION SET LEGACY_CARDINALITY_ESTIMATION = ON;

-- Example: Set MAXDOP to 4 for this database
ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = 4;
```

---

### 🌟 **Summary Points**

- **Database scoped configurations** provide granular control over database behavior.
- Great for tuning **multi-database servers** or **gradual upgrades**.
- Keep your **compatibility level** up-to-date to access the latest SQL performance features.
- Most options are set via T-SQL (`ALTER DATABASE` or `ALTER DATABASE SCOPED CONFIGURATION`).

---
