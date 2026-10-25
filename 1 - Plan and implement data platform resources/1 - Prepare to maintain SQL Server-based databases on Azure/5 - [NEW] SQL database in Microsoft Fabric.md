## 🧵 **What Is SQL Database in Microsoft Fabric?**

- A **developer-friendly OLTP database** that runs **inside a Microsoft Fabric workspace** (SaaS), not as an Azure resource.
- Built on the **same SQL Database Engine as Azure SQL Database**, so the T-SQL, query processing and most features are the same (compatibility level up to **170**).
- **Status: Generally Available** (announced at Microsoft Ignite, **Nov 2025**). Some sub-features, such as `ALTER DATABASE SET` options, full-text search and workspace-level private links, are still **preview**.
- Naming: the exam outline says **"Azure SQL Database in Microsoft Fabric"**. The product docs call it **"SQL database in Microsoft Fabric"** (or "Fabric SQL database"). They mean the same thing.

---

## 🔁 **The Killer Feature: Automatic Mirroring to OneLake**

- Every eligible table is **automatically replicated in near real time** to **OneLake** as **Delta (Parquet)** files. You don't need any ETL.
- Creating the database also creates a read-only **SQL analytics endpoint** on the mirrored data. Power BI, Spark, notebooks and warehouses can query it.
- **Cross-database three-part-name queries** across Fabric items (other SQL DBs, warehouses, lakehouses) run through the SQL analytics endpoint.
- Analytics runs on the **mirrored copy**, so it **doesn't load the OLTP engine**.

| Mirroring limitation                                                   | Detail                                                     |
| ---------------------------------------------------------------------- | ---------------------------------------------------------- |
| **Can't be disabled** (database level)                                 | Mirroring is always on; mirrors only to the same workspace |
| **Max 1,000 tables** mirrored per database                             | Extra tables are skipped                                   |
| Unsupported column types (`xml`, `geography`, `hierarchyid`, `sql_variant`, `rowversion`, UDTs, …) | Those columns aren't mirrored                              |
| Tables with **json / vector** columns, in-memory, Always Encrypted, graph, external tables | Not mirrored                                              |
| **RLS, object-level permissions, dynamic data masking**                | **Not propagated** to the OneLake copy                     |
| DDL blocked while mirrored                                             | **Switch/split/merge partition**, ALTER primary key, `REBUILD ... DATA_COMPRESSION = ROW/PAGE`, `ALTER INDEX ALL` |
| Clustered columnstore on an existing table                             | Not allowed while mirroring is running                     |

---

## ⚖️ **Fabric SQL Database vs Azure SQL Database**

| Area                         | Azure SQL Database                                      | SQL database in Fabric                                        |
| ---------------------------- | ------------------------------------------------------- | ------------------------------------------------------------- |
| Deployment / billing         | Azure resource; DTU or vCore; provisioned or serverless | Fabric item; billed from **Fabric capacity (CUs)**             |
| Scaling                      | Manual, or automatic in serverless                      | **Automatic** (up to **32 vCores**), auto-pause              |
| Max storage                  | 4 TB (GP/BC), 128 TB (Hyperscale)                        | **4 TB**; 150 DBs per workspace (3 on trial capacity)         |
| Authentication               | SQL auth + Microsoft Entra                              | **Microsoft Entra only**; no logins, no SQL auth              |
| Mirroring to OneLake         | Optional, set up manually                               | **Automatic, always on**                                       |
| Elastic pools                | Yes                                                     | No. A Fabric **capacity** plays a similar role               |
| Geo-replication / failover groups / geo-restore | Yes                                  | **No**                                                         |
| Backups                      | PITR 1–35 days + **LTR up to 10 years**                  | PITR **7 days default, up to 35**; **no LTR**; restore within the **same workspace** only |
| Always Encrypted / ledger / in-memory OLTP / CDC | Yes                                  | **No**                                                         |
| Elastic jobs / SQL Agent     | Elastic jobs                                            | Neither. Use **Data Factory pipelines** or **Airflow jobs**   |
| Networking                   | Firewall, VNet service endpoints, Private Link, Proxy/Redirect | Public by default; tenant/workspace private links; **Default** connection policy only; no VNet service endpoints |
| Encryption at rest           | TDE (service- or customer-managed key)                  | Service-managed by default; **workspace-level CMK** enables TDE |
| Auditing                     | Azure SQL auditing                                      | SQL auditing (Fabric)                                          |
| Tooling                      | Portal, PowerShell, CLI, Bicep/ARM                      | Fabric portal, **REST API/Fabric CLI**, SSMS, VS Code, git integration; **no Az PowerShell/CLI/Bicep** |
| Migration tool               | DMS / Azure SQL migration extension                     | **Fabric Migration Assistant for SQL database**                |

---

## 🎯 **When to Pick Which**

**Choose SQL database in Fabric when:**

- The org is **already on Fabric** and wants operational data **instantly available for analytics/Power BI** without ETL.
- **Translytical apps**: transactions plus near real-time analytics on the same data.
- **Reverse ETL**: push curated lakehouse/warehouse data back into an operational DB for apps or **GraphQL** APIs.
- **Operational data store (ODS)**: consolidate operational systems into one near real-time store.
- **AI apps / RAG**: vector data types and AI functions, close to your Fabric data.
- Teams want **minimal DBA effort**: autonomous scaling, auto-indexing and backups enabled by default.

**Choose Azure SQL Database (or MI) when you need:**

- **SQL authentication**, Always Encrypted, ledger, CDC or In-Memory OLTP.
- **Geo-DR**: active geo-replication, failover groups, geo-restore, LTR backups.
- **VNet integration / service endpoints**, Proxy or Redirect connection policy, fine-grained Azure networking.
- **> 4 TB** or more than 32 vCores (Hyperscale), elastic pools, elastic jobs.
- Infrastructure-as-code deployment with **ARM/Bicep/PowerShell/CLI**.

> 💡 You can get OLTP-to-OneLake replication **without** moving the DB: **Fabric mirroring** of an existing Azure SQL Database or Managed Instance. On MI this needs the **SQL Server 2025 or Always-up-to-date** update policy.

---

## 🛡️ **DBA View: What You Still Manage**

```sql
-- Microsoft Entra users only (no CREATE LOGIN).
-- Share the database first: users need the Fabric "Read" item permission to connect.
CREATE USER [dba-team@contoso.com] FROM EXTERNAL PROVIDER;
ALTER ROLE db_owner ADD MEMBER [dba-team@contoso.com];

-- Backup history (same DMV as Azure SQL Database)
SELECT backup_type, backup_start_date, backup_finish_date
FROM sys.dm_database_backups
WHERE in_retention = 1
ORDER BY backup_finish_date DESC;

-- Check TDE state after a workspace customer-managed key is configured
SELECT DB_NAME(database_id) AS db, encryption_state_desc, encryptor_type
FROM sys.dm_database_encryption_keys
WHERE database_id <> 2;
```

- **Access**: Fabric workspace roles + item **sharing** (needs at least **Read** item permission to connect), plus normal SQL database roles, **RLS** and **OLS** inside the database.
- **Backups**: full weekly, differential every 12 h, log about every 10 min, on **ZRS** storage. You can't change the schedule. Restore (PITR, deleted DB) via the **Fabric portal**.
- **Monitoring**: Performance Dashboard, Query Store/DMVs, Capacity Metrics app, Database Hub (preview).
- **Encryption**: storage encryption with Microsoft-managed keys by default. A **workspace CMK** automatically enables **TDE** with that key (RSA 2048/3072 only). If the key is lost for **more than 30 min**, the DB can't auto-heal.
- ⚠️ The Fabric limitations page still says "CMK/TDE not supported", but the encryption and overview pages describe **workspace-level CMK with TDE**. Treat **workspace CMK** as the current answer.

---

## ✅ **Summary**

- **SaaS OLTP database in Fabric** on the Azure SQL Database engine. **GA since Nov 2025.**
- **Automatic near real-time mirroring to OneLake (Delta)** + a SQL analytics endpoint = analytics with **no ETL**.
- **Entra-only auth**, auto-scale up to 32 vCores / 4 TB, PITR up to 35 days. No geo-DR, LTR, elastic pools, Always Encrypted, ledger or CDC.
- Best for **translytical**, **reverse ETL**, **ODS** and **AI/RAG** apps in Fabric-centric organizations.

---

## 🎯 **Exam Tips / Traps**

- "OLTP data must be **queryable by Power BI/Spark in near real time** with **least effort/no ETL**" → **SQL database in Fabric** (or Fabric mirroring of an existing Azure SQL DB).
- "Must use **SQL authentication**" or "needs **Always Encrypted / ledger**" → **not** Fabric; use Azure SQL Database.
- "Cross-region **failover group** / **geo-restore** / **10-year LTR**" → **Azure SQL Database**. Fabric SQL DB has none of these.
- "Mask/RLS-protect data for analysts" → RLS/DDM **aren't propagated** to the OneLake mirror. Secure the analytics side separately.
- "Schedule maintenance T-SQL" → no SQL Agent or elastic jobs in Fabric; use **Data Factory pipelines**.
- Don't confuse the **Fabric SQL database** (read/write OLTP) with the **SQL analytics endpoint** (read-only) or **Fabric Warehouse**.
- Elastic pool equivalent in Fabric = **Fabric capacity** shared across workspaces.

> Source: https://learn.microsoft.com/en-us/fabric/database/sql/overview · https://learn.microsoft.com/en-us/fabric/database/sql/limitations · https://learn.microsoft.com/en-us/fabric/database/sql/mirroring-limitations · https://learn.microsoft.com/en-us/fabric/database/sql/encryption · https://learn.microsoft.com/en-us/fabric/database/sql/backup · https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/update-policy · https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/dp-300 · https://blog.fabric.microsoft.com/en-US/blog/announcing-sql-database-in-fabric-is-now-generally-available-ga/
