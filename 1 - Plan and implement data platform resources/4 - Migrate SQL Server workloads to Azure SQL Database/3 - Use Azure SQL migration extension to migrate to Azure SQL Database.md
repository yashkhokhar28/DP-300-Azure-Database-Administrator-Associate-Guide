## 📦 **Context Example**

In the example scenario, the company wants to migrate their HR database over the weekend (when the system is less busy). They plan **an offline migration**, meaning the database will be taken offline during the move.

---

## 🛠️ **Azure SQL Migration Extension Overview**

The **Azure SQL Migration extension for Azure Data Studio** helps you:

- Assess your databases for migration readiness
- Recommend the best Azure SQL service
- Migrate the data
- Handle **small to medium** databases efficiently
- Use Azure Database Migration Service (DMS) behind the scenes for the actual transfer
- Migrate multiple databases **for free**

---

## 🔑 **Step-by-Step Migration Process**

| **Step**   | **Description**                                                                                                    |
| ---------- | ------------------------------------------------------------------------------------------------------------------ |
| **Setup**  | Install Azure Data Studio + Azure SQL Migration extension. Start the **Migration wizard**.                         |
| **Step 1** | Select the databases you want to assess and migrate.                                                               |
| **Step 2** | Run a migration assessment. It shows compatibility or feature gaps, and recommends Azure SQL SKUs (service tiers). |
| **Step 3** | Connect to your Azure SQL Database target using your Azure account.                                                |
| **Step 4** | Select/create the **Azure Database Migration Service (DMS)** instance.                                             |
| **Step 5** | Configure data source:                                                                                             |

- Connect to your on-prem SQL Server.
- Select which tables to migrate.
- Make sure the schema already exists on the target (use a `.dacpac`, SQL Projects, or DMA). |
  \| **Step 6** | Review and start the migration.

➡️ **Note:** The service skips empty tables during migration to speed up the process.

---

## ⚠️ **Important Limitation**

Azure SQL Database **doesn't support migrating table names with double-byte characters (e.g., some non-English characters).**
You must rename those tables before migration and rename them back after completion.

---

## 🔍 **Migration Statuses**

You’ll see these status stages in the extension:

- **Preparing for copy:** Disabling autostats, triggers, and indexes on target
- **Copying:** Moving data from source to target
- **Copy finished:** Waiting for all tables to finish copying
- **Rebuilding indexes:** Rebuilding indexes on the target
- **Succeeded:** Migration is complete

---

## 🚨 **Performance Considerations**

- Scale up your Azure SQL Database compute (vCores) temporarily to speed up migration.
- The machine running the **self-hosted integration runtime** should have enough CPU/memory.
- Azure Data Factory (used in the background) may slow down when migrating **many small tables**.
- Large **BLOB columns** may timeout—this can cause failures.
- Recommended: No more than **10 concurrent database migrations** per integration runtime node. Scale out horizontally if needed.

---

## 📊 **Monitoring Migration Progress**

You can track migration status in two places:

### In Azure Data Studio

- View "Migration dashboard"
- Check in-progress, completed, or failed migrations

### In Azure Portal

- Go to your Azure Database Migration Service (DMS) resource
- Monitor active migrations in the **"Monitor migrations"** section

---

## ⚙️ **Migration at Scale (PowerShell / Azure CLI)**

You can automate migrations using PowerShell or CLI.

Example: Migrate the full **AdventureWorks** database:

```powershell
New-AzDataMigrationToSqlDb `
-ResourceGroupName MyGroup `
-SqlDbInstanceName myserver `
-Kind "SqlDb" `
-TargetDbName AdventureWorks `
-SourceDatabaseName AdventureWorks `
...
```

Example: Migrate **only selected tables**:

```powershell
New-AzDataMigrationToSqlDb `
...
-TableList "[Person].[Person]", "[Person].[EmailAddress]" `
...
```

---

## ✅ **Summary**

- Azure SQL Migration extension is a **guided, easy-to-use** option for migrating to Azure SQL Database.
- It uses DMS in the background but simplifies the workflow.
- Best for **offline migration** of small/medium databases.
- Supports assessment + migration in a single flow.
- Scale compute + prepare schema beforehand for smoother migration.

---
