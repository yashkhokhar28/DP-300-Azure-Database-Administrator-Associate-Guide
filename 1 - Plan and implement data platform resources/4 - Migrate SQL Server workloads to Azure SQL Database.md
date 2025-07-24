# 🧠 Azure SQL Database Migration – Full Summary with Real-World Context

## 🔷 What Is Azure SQL Database?

Azure SQL Database is a **cloud-based service** (Platform as a Service – PaaS) from Microsoft. Unlike traditional SQL Server, **you don’t manage the OS, backups, or hardware** — Azure does all that for you.

### 🎯 Real-World Example:

Imagine you run a **bike manufacturing company** with databases for products, inventory, and HR. Instead of maintaining physical servers, you move them to Azure SQL — reducing effort, cost, and getting better reliability.

---

## ✅ When to Use Azure SQL Database (Instead of Managed Instance)

| Scenario                                | Why Azure SQL is Better                                |
| --------------------------------------- | ------------------------------------------------------ |
| Unpredictable workloads                 | It auto-scales resources, saving money.                |
| New apps with unknown performance needs | Start small and grow later.                            |
| Simpler setup preferred                 | Easier for small teams.                                |
| Need lots of storage per database       | Azure SQL supports more storage than Managed Instance. |

---

## 🚦 Migration Process Overview

1. **Assess** your databases (use DMA tool).
2. **Create** Azure SQL Database.
3. Choose **online** (no downtime) or **offline** (with downtime) migration.
4. **Migrate** using tools like DMA or Azure DMS.
5. **Optimize** your database post-migration.

---

## 🛡️ Why Assessment Is Crucial

Before migration, **run an assessment** to:

- Find out what won’t work in Azure SQL.
- Discover performance improvements you can gain.

📌 Tool: Use **Data Migration Assistant (DMA)** for this.

---

## 💎 Benefits of Azure SQL Database

| Category          | Features                                                            |
| ----------------- | ------------------------------------------------------------------- |
| Backup            | Auto backups, point-in-time restore, long-term retention (10 years) |
| High Availability | 99.99% uptime, zone redundancy                                      |
| Security          | Encryption (TDE), advanced threat protection, row-level security    |
| Performance       | Scale resources, elastic pools, sharding                            |
| Licensing         | Flexible pricing (DTU/vCore), Azure Hybrid Benefit                  |

---

## 🌟 Azure-Only Features (Not in Managed Instance)

- **Hyperscale**: Huge databases with separate compute and storage scaling.
- **Serverless Auto-Scale**: Adjusts resources based on need.
- **Elastic Query**: Query across databases.
- **Automatic Tuning**: Azure manages indexes for better performance.
- **Elastic Jobs**: Schedule tasks across multiple databases.
- **Query Insights**: Analyze and fix slow queries.

---

## 🔁 Migration Tools & Modes

| Tool                           | Type    |
| ------------------------------ | ------- |
| Azure DMS                      | Offline |
| Transactional Replication      | Online  |
| Azure Migrate                  | Offline |
| Import/Export Wizard / BACPAC  | Offline |
| Bulk Copy (`bcp`)              | Offline |
| Azure Data Factory             | Offline |
| DMA (Data Migration Assistant) | Offline |

📌 **Tip**: Use **Azure DMS** for large, complex migrations.

---

## 🚀 Performance Tips During Migration

- Fix I/O bottlenecks before migration.
- Use high-tier DB during migration (e.g., Hyperscale).
- Disable auto-stat updates temporarily.
- Scale down post-migration to save money.
- Place files near the target region.
- Retry failed connections using **exponential backoff** (e.g., 5s → 10s → 20s…).

---

## 🧪 Retry Logic in Action

If a SELECT query fails:

1. Wait 5 seconds.
2. Try again.
3. Increase wait time if needed.
4. Always retry using a new connection.

---

## 🧠 Real Example: Weekend HR DB Migration

A bike company migrates its **HR database** over the weekend using **offline migration**, minimizing disruption and saving costs.

---

## 🧰 Azure SQL Migration Extension (for Azure Data Studio)

This free tool helps:

- Assess databases
- Recommend best Azure options
- Migrate small/medium databases
- Use Azure DMS in the background

### Migration Steps:

1. Install Azure Data Studio + extension.
2. Select databases and run assessment.
3. Connect to Azure SQL.
4. Create/select Azure DMS.
5. Configure data source and schema.
6. Start migration (skips empty tables to save time).

---

## ⚠️ Limitations

- Table names with **double-byte characters** (like some non-English names) must be renamed before migrating.

---

## 📊 Migration Progress Monitoring

You can monitor from:

- **Azure Data Studio** → Migration dashboard
- **Azure Portal** → Monitor DMS progress

---

## ⚙️ Scale with PowerShell / Azure CLI

You can use scripts to automate bulk migrations.
Example:

```powershell
New-AzDataMigrationToSqlDb -TargetDbName AdventureWorks
```

---

## 🛠️ What is Data Migration Assistant (DMA)?

A free tool to:

- Check for compatibility issues.
- Generate migration scripts.
- Move schema and data (good for small/medium databases).

### Types of Migrations in DMA:

| Type          | Description                           |
| ------------- | ------------------------------------- |
| Schema + Data | Structure + data                      |
| Schema only   | Only tables/views/procedures          |
| Data only     | Only data (schema must already exist) |

---

## 📋 Step-by-Step with DMA

1. Run assessment → fix issues.
2. Create project → connect to source SQL Server.
3. Choose objects → tables, views, etc.
4. Deploy schema to Azure.
5. Migrate data.
6. Monitor progress.

🔧 Optional: Tune config via `dma.exe.config` (e.g., parallel DBs, timeouts).

---

## ✅ DMA Best Practices

- Don’t run DMA on your production server.
- Use a shared folder for temporary files.
- Enable encryption.
- Fix constraints before migration.

---

## 📦 What’s a BACPAC File?

A compressed `.bacpac` archive of:

- Schema (structure)
- Data (rows)

Best for **offline, one-time migrations**.

---

### How to Use BACPAC

1. Export `.bacpac` from SSMS or SqlPackage.
2. Upload to Azure Blob or local storage.
3. Import via Azure Portal or SqlPackage.

Example command:

```bash
SqlPackage.exe /a:import /sf:AdventureWorks.bacpac
```

---

## ⚠️ BACPAC Limitations

- Only works offline (requires downtime).
- Doesn’t include logins, jobs, linked servers.
- Not ideal for databases >200 GB.

---

## 🔁 Transactional Replication (For Online Migrations)

Used when apps **must stay live**.

### How It Works:

1. Snapshot initial data.
2. Sync changes using SQL Server logs.
3. Push changes to Azure in real time.

📌 Azure supports **Push subscriptions** only.

---

### Key Components:

| Term         | Role                            |
| ------------ | ------------------------------- |
| Publisher    | Source SQL Server               |
| Subscriber   | Azure SQL Database              |
| Distributor  | Coordinates replication         |
| Article      | Table/view being replicated     |
| Publication  | A group of articles             |
| Subscription | Request for data from Azure SQL |

---

### Setup Steps

1. Configure distributor (`sp_adddistributor`)
2. Enable publisher (`sp_addpublication`)
3. Add tables (`sp_addarticle`)
4. Create subscription (`sp_addsubscription`)
5. Push data (`sp_addpushsubscription_agent`)

✅ Monitor using SQL Server Management Studio (not Azure Portal).

---

## ⚠️ Replication Limitations

- Only **Transactional Replication** is supported (no merge, no peer-to-peer).
- Schema changes during replication are limited.
- SQL Agent jobs must be managed manually.

---

## 🎯 When to Use Replication?

| Use Case                           | Replication Fit        |
| ---------------------------------- | ---------------------- |
| 24/7 critical system               | ✅ Yes                 |
| One-time migration (with downtime) | ❌ No (use DMS/BACPAC) |

---

## 📦 Partial Data Migration – Why and How?

### Why Migrate Partially?

- Keep **sensitive data** on-premises.
- Move only **active or useful data**.
- Save on **cloud storage costs**.

### Example:

The bike company migrates only **products & customers** to Azure, while keeping **sales data** on-premises.

---

### Tools for Partial Migration

#### 1. `bcp` Utility

Simple tool to copy data one table at a time.

Example:

```bash
bcp AdventureWorks.dbo.Product out product_data.txt
```

✅ Fast, scriptable
⚠️ Schema must exist beforehand.

#### 2. Azure Data Factory (ADF)

Cloud-based tool for advanced, scheduled migrations.

✅ Graphical interface, filter data, automate jobs
⚠️ Requires setup + integration runtime for on-prem access.

---

## 🧭 Which Tool to Use?

| Situation                     | Best Tool                                |
| ----------------------------- | ---------------------------------------- |
| Quick table copy              | `bcp`                                    |
| Scheduled/filtered migrations | Azure Data Factory                       |
| Full schema + data migration  | Azure Migration Extension / DMA / BACPAC |

---

## 🎯 Final Summary

Azure SQL Database gives you:

- **PaaS convenience** (no server mgmt)
- **Elastic scale**
- **Built-in security, backups, HA**
- **Multiple migration paths** (DMA, DMS, BACPAC, replication)

Choose tools based on:

- **Downtime allowance**
- **Database size**
- **Data sensitivity**
- **Complexity of schema**

---
