## 🛠️ **What is Data Migration Assistant (DMA)?**

**DMA** is a free Microsoft tool that helps:

- Assess SQL Server databases for compatibility issues with Azure SQL.
- Generate the necessary scripts to migrate schema.
- Transfer schema and data from SQL Server to Azure SQL Database.

DMA is best for **smaller, straightforward migrations**, but for **large migrations**, it’s better to use **Azure Database Migration Service (DMS).**

---

## 🔄 **Migration Types Supported in DMA**

| Migration Type      | What it Includes                                                                                |
| ------------------- | ----------------------------------------------------------------------------------------------- |
| **Schema and data** | Migrates the database structure **AND** the data.                                               |
| **Schema only**     | Migrates only tables, views, procedures, etc. No data.                                          |
| **Data only**       | Migrates data only. You must manually create the schema on Azure SQL before migrating the data. |

---

## 🔍 **Step-by-Step Migration Process**

### 1. **Assess the Database**

- Run a **compatibility assessment** on your source database.
- Identify issues like deprecated features, syntax problems, or missing functions.
- Apply fixes manually using T-SQL or upload the assessment report to **Azure Migrate** for further planning.

---

### 2. **Create a Migration Project**

- Choose:

  - **Source type:** SQL Server
  - **Target type:** Azure SQL Database

- Connect to your **source SQL Server** using the required authentication.

---

### 3. **Select Databases & Schema Objects**

- Select the database you want to migrate.
- On the **"Select objects" tab**, choose which tables, views, and procedures you want to include.
- DMA will highlight objects that can't be migrated as-is and suggest fixes.

---

### 4. **Generate & Deploy Schema**

- Generate a **SQL deployment script** for the schema.
- Review the script and deploy it to the target Azure SQL Database.

---

### 5. **Migrate the Data**

- Once the schema is deployed, select the tables whose data you want to migrate.
- Start the data migration process.

---

### 6. **Monitor the Migration**

- DMA displays the **progress and status** of the migration.
- You can filter to view successful, in-progress, or failed migrations.

---

## ⚙️ **Performance Tuning in dma.exe.config**

You can fine-tune DMA by modifying its configuration file:

| Setting             | Purpose                                                   |
| ------------------- | --------------------------------------------------------- |
| `parallelDatabases` | Adjusts how many databases migrate in parallel.           |
| `ConnectionTimeout` | Sets how long to wait before timing out a SQL connection. |

Example location of the config file:
`%ProgramFiles%\Microsoft Data Migration Assistant\dma.exe.config`

---

## ✅ **Best Practices**

| Practice                                                      | Why it's Important                          |
| ------------------------------------------------------------- | ------------------------------------------- |
| Don’t run DMA on the source SQL Server                        | Prevents overloading your production server |
| Use a **shared network folder** for temporary files           | Avoids unnecessary file copies              |
| Set correct permissions on the shared folder                  | Prevents access errors during migration     |
| Enable **encrypted connections**                              | Secure your data during migration           |
| Check and fix **untrusted constraints** before migrating data | Prevents performance issues in query plans  |

---

## ⚠️ **Key Notes**

- For large databases, DMA is slower compared to Azure Database Migration Service (DMS).
- Increase Azure SQL Database’s performance tier (e.g., to **P15**) temporarily during migration for faster data import.
- DMA doesn’t automatically create Azure SQL resources—you must set up your Azure SQL Database before migrating.

---

## 📌 **Next Step**

The next learning unit focuses on **migrating using BACPAC files**, another offline migration method.

---
