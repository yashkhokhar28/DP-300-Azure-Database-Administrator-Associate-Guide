## 📦 **Context Example**

- The bike company's **HR database** is business-critical but rarely used at weekends → plan an **offline** migration from Friday evening to Monday morning.
- Pre-migration checks (feature and compatibility issues) are assumed done with **Azure Migrate**.
- This unit **replaces** the old "Azure SQL migration extension" and "Data Migration Assistant" units (both tools are retired). The migration engine is the same: **Azure Database Migration Service (DMS)**.

---

## 🛠️ **Azure Database Migration Service (DMS): What You Need to Know**

- A **fully managed** Azure service that migrates many database sources to Azure data platforms.
- To **Azure SQL Database**, DMS migrations are **offline** (downtime starts when the copy starts). For online to SQL Database, use **transactional replication**.
- You can drive it from the **Azure portal**, **Azure CLI** (`az datamigration` extension) or **PowerShell** (`Az.DataMigration`). Scripts are ideal for:
  - **CI/CD pipelines**
  - Migrating **many databases at scale**
  - **Repeatable** migrations

---

## ✅ **Prerequisites**

| #   | Requirement                                                                                 |
| --- | ------------------------------------------------------------------------------------------- |
| 1   | **Azure CLI** with the `datamigration` extension (`az extension add --name datamigration`) |
| 2   | A **DMS** resource in your subscription                                                     |
| 3   | The **target Azure SQL Database** provisioned (schema deployed, or use schema migration)    |
| 4   | A **self-hosted integration runtime (SHIR)** that can reach the source SQL Server           |

---

## 🔑 **Step-by-Step with Azure CLI**

### 1. Create the DMS resource

```bash
az datamigration sql-service create \
    --resource-group "<rg>" \
    --sql-migration-service-name "<dms-name>" \
    --location "<region>"
```

### 2. Migrate the schema (source → target)

```bash
az datamigration sql-server-schema \
    --action "MigrateSchema" \
    --src-sql-connection-str "Server=<src>;Initial Catalog=<srcDb>;User ID=<user>;Password=<pwd>" \
    --tgt-sql-connection-str "Server=<srv>.database.windows.net;Initial Catalog=<tgtDb>;User ID=<user>;Password=<pwd>"
```

### 3. Start the data migration

```bash
az datamigration sql-db create \
    --resource-group "<rg>" \
    --sqldb-instance-name "<target-server>" \
    --target-db-name "<tgtDb>" \
    --source-database-name "<srcDb>" \
    --source-sql-connection authentication="SqlAuthentication" data-source="<src>" user-name="<user>" password="<pwd>" encrypt-connection=true trust-server-certificate=true \
    --target-sql-connection authentication="SqlAuthentication" data-source="<srv>.database.windows.net" user-name="<user>" password="<pwd>" encrypt-connection=true trust-server-certificate=true \
    --scope "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Sql/servers/<target-server>" \
    --migration-service "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.DataMigration/sqlMigrationServices/<dms-name>"
```

- **Only some tables?** Add `--table-list "[Person].[Person]" "[Person].[EmailAddress]"`.
- DMS **skips empty tables**, even if you select them.

---

## 📊 **Monitor, Wait, Cancel (CLI)**

| Task                    | Command                                                                                                      |
| ----------------------- | ------------------------------------------------------------------------------------------------------------ |
| Check status + errors   | `az datamigration sql-db show -g <rg> --sqldb-instance-name <srv> --target-db-name <db> --expand "MigrationStatusDetails"` |
| Pause script until done | `az datamigration sql-db wait ... --created`                                                                 |
| Cancel a migration      | `az datamigration sql-db cancel ... --migration-operation-id <id>`                                          |

**Azure portal:** open the **DMS** resource → **Monitor migrations**. When the status is **Succeeded**, validate the schema and data on the target.

### 🔍 Migration statuses

- **Preparing for copy**: disabling autostats, triggers and indexes on the target table
- **Copying**: data copy in progress
- **Copy finished**: waiting for the other tables to finish
- **Rebuilding indexes**: rebuilding indexes on the target tables
- **Succeeded**: all data copied and indexes rebuilt

---

## ⚙️ **Same Migration with PowerShell**

```powershell
New-AzDataMigrationToSqlDb `
  -ResourceGroupName "<rg>" -SqlDbInstanceName "<target-server>" -Kind "SqlDb" `
  -TargetDbName "<tgtDb>" -SourceDatabaseName "<srcDb>" `
  -SourceSqlConnectionAuthentication SQLAuthentication -SourceSqlConnectionDataSource "<src>" `
  -SourceSqlConnectionUserName "<user>" -SourceSqlConnectionPassword $sourcePass `
  -TargetSqlConnectionAuthentication SQLAuthentication -TargetSqlConnectionDataSource "<srv>.database.windows.net" `
  -TargetSqlConnectionUserName "<user>" -TargetSqlConnectionPassword $targetPass `
  -Scope "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Sql/servers/<target-server>" `
  -MigrationService "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.DataMigration/SqlMigrationServices/<dms-name>"
# Subset of tables: add  -TableList "[Person].[Person]", "[Person].[EmailAddress]"
```

---

## 🚨 **Performance Considerations**

- Speed depends on the **target SKU** and the **SHIR host** → **scale up** the Azure SQL Database before you start, scale down after.
- The SHIR machine must handle the **CPU and memory** load of the copy.
- **Many tables** = slow (Azure Data Factory starts an activity per table, even for small tables).
- Tables with **large BLOB columns** can **time out**.
- Up to **10 concurrent database migrations per SHIR node**. Scale out the SHIR or add more machines for more.

---

## ✅ **Summary**

- Current tool for **offline** migration to Azure SQL Database = **Azure DMS** (portal, `az datamigration`, `New-AzDataMigrationToSqlDb`).
- Flow: **create DMS → migrate schema → `sql-db create` → monitor (`show` / portal) → validate**.
- Needs a **SHIR**; the target DB must exist.
- Online requirement → **transactional replication**, not DMS.

---

## 🎯 **Exam Tips / Traps**

- "Script the migration of many databases to Azure SQL Database in a pipeline" → **`az datamigration sql-db create`** or **`New-AzDataMigrationToSqlDb`**.
- "Migrate only certain tables" → **`--table-list` / `-TableList`**.
- "Some selected tables weren't migrated" → they were **empty** (DMS skips them).
- "Minimal downtime to Azure SQL Database" → **not** DMS (offline only) → **transactional replication**.
- "Where to watch progress in the portal?" → DMS resource → **Monitor migrations**.

> Source: https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-sql-databases/3-migrate-using-azure-migration-extension · https://learn.microsoft.com/en-us/cli/azure/datamigration · https://learn.microsoft.com/en-us/powershell/module/az.datamigration · https://learn.microsoft.com/en-us/azure/dms/dms-overview
