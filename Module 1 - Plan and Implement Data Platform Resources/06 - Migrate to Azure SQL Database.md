# Migrate to Azure SQL Database

> **Exam skills:** Implement an offline migration strategy, Implement an online migration strategy, Implement a migration to Azure, Implement a migration between Azure SQL services
> **Reading time:** ~8 min

## The big idea
Azure SQL Database can't restore a normal `.bak` file. So you move **schema and data**, not backups. Offline: use **DMS** or a **BACPAC**. Online: use **transactional replication**. Only some tables: use **bcp** or **Azure Data Factory**.

Analogy: you can't carry the whole fridge into a hotel room. You unpack the food and carry it in item by item.

## Azure Database Migration Service (DMS)
DMS is the main managed migration engine. To SQL Database it is **offline only**: downtime starts when the copy starts. Behind the scenes it runs **Azure Data Factory** pipelines.

You can drive it from the **Azure portal**, **Azure CLI** (`az datamigration`) or **PowerShell** (`Az.DataMigration`). Scripts are good for pipelines, many databases and repeat runs.

**What you need:**
1. A **DMS** resource.
2. The **target database** already created.
3. A **self-hosted integration runtime (SHIR)**: a small agent installed on a machine that can reach the source. It also does the copy work, so size it well.
4. **Permissions:** source login = **db_datareader + VIEW ANY DEFINITION**. Target login = **db_owner**.
5. The **schema** on the target. Use DMS schema migration (tick **Migrate missing schema**), or deploy a DACPAC with SqlPackage first.

**Steps with Azure CLI:**
```bash
az datamigration sql-service create -g rg --sql-migration-service-name dms1 --location westeurope
az datamigration sql-server-schema --action MigrateSchema \
   --src-sql-connection-str "<source>" --tgt-sql-connection-str "<target>"
az datamigration sql-db create -g rg --sqldb-instance-name tgtsrv --target-db-name HR \
   --source-database-name HR --source-sql-connection ... --target-sql-connection ... \
   --scope "<target server id>" --migration-service "<dms id>"
az datamigration sql-db show -g rg --sqldb-instance-name tgtsrv --target-db-name HR --expand MigrationStatusDetails
```
- Only some tables? Add `--table-list` (PowerShell: `-TableList`).
- PowerShell equivalent: `New-AzDataMigrationToSqlDb`.
- In the portal, watch progress at **DMS resource → Monitor migrations**.

**Status flow:** Preparing for copy (disables autostats, triggers, indexes on the target) → Copying → Copy finished → Rebuilding indexes → **Succeeded**.

**Speed tips:**
- **Scale up** the target before, scale down after.
- **Many small tables** are slow, because ADF starts one activity per table.
- **Large BLOB columns** can time out.
- Up to **10 concurrent database migrations per SHIR node**. Add nodes for more.

**Example:** A bike company's HR database is quiet at weekends. The DBA starts a DMS offline migration on Friday night, checks Monitor migrations on Saturday, and switches the app on Sunday.

Note: the old **Azure SQL migration extension** (Azure Data Studio, retired 28 Feb 2026) and **DMA** (retired July 2025) did the same job with a wizard. DMA had three modes: schema and data, schema only, data only. The engine today is DMS.

## BACPAC (export and import)
A **BACPAC** is one compressed file with **schema and data**.
1. **Export** from the source with SSMS or **SqlPackage**.
2. Upload to **Azure Blob Storage**.
3. **Import** with the **portal** (logical server → **Import database**) or **SqlPackage**.

```bash
SqlPackage /a:Import /sf:HR.bacpac \
  /tcs:"Data Source=tgtsrv.database.windows.net;Initial Catalog=HR;User Id=admin;Password=<pwd>" \
  /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P6
```

Limits:
- **Offline** only.
- **No logins, Agent jobs or linked servers** (database-level objects only).
- Databases **over 150 GB** often fail in the portal → use **SqlPackage**.
- Portal import creates a **new single database** only (not into a pool). The storage can't sit behind a firewall.
- For **Managed Instance**, import with **SqlPackage** or SSMS, not the portal.
- Watch progress in **Import/Export history**.

**Example:** A 20 GB club membership database moves over a weekend. Export a BACPAC, upload it, import it at P6, then scale down.

## Transactional replication (online)
The only **online** way into SQL Database. The source keeps working while changes flow across.

| Role | Who |
|------|-----|
| **Publisher** | Source SQL Server (2016+) or Managed Instance |
| **Distributor** | Moves the changes. Often on the publisher. |
| **Subscriber** | Azure SQL Database (**push subscription only**) |
| **Article** | One table |
| **Publication** | A set of articles |

How it works: a **Snapshot Agent** copies the starting data. The **Log Reader Agent** reads changes from the log. The **Distribution Agent** pushes them to Azure.

Set up with `sp_adddistributor`, `sp_addpublication`, `sp_addarticle`, `sp_addsubscription`, `sp_addpushsubscription_agent`.

Rules:
- Every table needs a **primary key**.
- SQL Database can only be a **push subscriber** of **transactional or snapshot** replication.
- Connect with **SQL authentication** (Entra auth is also possible in some cases).
- Monitor in **SSMS** on the publisher, **not** in the Azure portal.

**Cutover:** stop app writes → check all changes arrived → point the app to Azure → remove replication.

**Example:** A warehouse inventory system runs 24/7. Replication keeps Azure in sync for a week. Cutover takes five minutes on a Sunday morning.

## Moving only some data
| Need | Tool |
|------|------|
| Quick one-time copy of a table | **bcp** (bulk copy; target table must exist) |
| Filtered, scheduled or transformed loads | **Azure Data Factory** (needs a SHIR for on-premises) |

```bash
bcp Shop.dbo.Product out product.dat -S onpremsrv -T -n
bcp Shop.dbo.Product in product.dat -S tgtsrv.database.windows.net -U admin -P <pwd> -n
```

**Example:** A company keeps sensitive sales data on-premises. It moves only the customer and product tables to Azure with Data Factory.

## Between Azure SQL services
- **SQL Database → SQL Database or MI:** BACPAC export/import (SqlPackage for MI).
- **MI → SQL Database:** MI can be a **publisher** for transactional replication, so you can move online. Or use BACPAC offline.
- **Selected tables, any direction:** Azure Data Factory or bcp.
- **MI → MI:** database **copy/move** (lesson 7).

## Known DMS limits to SQL Database
- **Computed columns** are not migrated.
- Columns with defaults may get the **default instead of NULL**.
- **Empty tables are skipped.**
- **Double-byte characters** in table names are not supported. Rename, migrate, rename back.
- **Disable CDC on the source** before schema migration.

## Remember for the exam
- DMS to SQL Database = **offline only**. Online = **transactional replication**.
- Script many migrations → `az datamigration sql-db create` or `New-AzDataMigrationToSqlDb`.
- Only some tables in DMS → `--table-list` / `-TableList`.
- "Some tables didn't migrate" → they were **empty**.
- BACPAC over 150 GB → **SqlPackage**, not the portal.
- BACPAC never moves logins or jobs.
- Replication to SQL Database = **push** subscription, tables need a **primary key**, monitor in **SSMS**.

## Quick check
1. You must migrate 40 databases to SQL Database from a CI/CD pipeline. What do you use?
   <details><summary>Answer</summary>Azure DMS driven by Azure CLI (`az datamigration`) or PowerShell (`New-AzDataMigrationToSqlDb`).</details>
2. A portal BACPAC import of a 300 GB database keeps failing. What do you do?
   <details><summary>Answer</summary>Import with SqlPackage. The portal often fails over 150 GB.</details>
3. Replication setup fails on one table. What is the likely cause?
   <details><summary>Answer</summary>The table has no primary key. Transactional replication needs one.</details>
