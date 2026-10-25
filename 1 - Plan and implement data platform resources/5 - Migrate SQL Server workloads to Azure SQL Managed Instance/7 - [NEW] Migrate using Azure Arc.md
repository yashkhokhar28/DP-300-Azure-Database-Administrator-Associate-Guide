## 🌐 **What Is SQL Server Migration in Azure Arc?**

- When a SQL Server instance is **enabled by Azure Arc**, the **Azure portal** becomes a single place to **assess, pick a target, migrate and cut over** to **Azure SQL Managed Instance**.
- Portal path: Arc-enabled **SQL Server instance → Migration → Database migration**.
- Under the hood it uses the two MI migration engines: **Managed Instance link** or **Log Replay Service (LRS)**.
- This is the portal replacement for the retired **Azure Data Studio migration extension** (MI scenarios).

> Scenario: the bike company's distribution team has business-critical databases that need **minimal downtime**, and wants a **simple, portal-based** migration while still choosing the method.

---

## 🔁 **The 4 Stages (tiles on the Database migration pane)**

| #   | Stage                     | What happens                                                                                              |
| --- | ------------------------- | --------------------------------------------------------------------------------------------------------- |
| 1   | **Assess source instance** | Readiness report: feature compatibility, migration **blockers**, **right-sized** MI tier/config recommendation |
| 2   | **Select target**          | Pick an existing MI or **create** a new one from the pane                                                 |
| 3   | **Migrate data**           | Choose **real-time replication (MI link)** or **log shipping (LRS)**, select databases, start              |
| 4   | **Monitor and cutover**    | Track progress (status, duration, replication **lag**) → **Ready for cutover** → **Cutover** (or cancel)  |

- **Discovery + assessment run automatically every weekend**. You can also start them manually at any time (takes minutes, no extra setup).
- The pane also shows a **summary**: total databases, recommended target, completed and in-progress migrations.
- **Microsoft Copilot** is built in: compare methods, explain the assessment, start, monitor and complete the migration.

---

## ⚖️ **MI Link vs LRS in the Arc Experience**

| -                       | **MI link** (real-time replication)                                   | **LRS** (continuous backup/restore)                                     |
| ----------------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Technology              | **Distributed availability group**                                    | **Log shipping** via an intermediary **Azure Blob Storage** account     |
| Target during migration | **Read-only** DB on MI (offload reads, check data)                    | No readable copy (DB in restoring state)                                |
| Supported versions      | SQL Server **2016+** on Windows Server 2016+                          | SQL Server **2012+** on Windows Server 2012+                            |
| Editions                | Enterprise, Standard, Developer                                       | **All** editions                                                        |
| Simultaneous migrations | Up to **10 DBs** (Azure extension for SQL Server **1.1.3348.364+**), else 1 | **Unlimited**                                                     |
| Recommended for         | Business-critical, **minimum downtime**, GP **or Business Critical** target | General Purpose target where some planned downtime is OK (not recommended for BC) |

- **LRS in Arc:** you must first **upload at least one full backup** to the blob container (and set permissions). Keep uploading log backups to the same container until cutover.
- **MI link in Arc:** each link takes up to **~10 minutes** to create. **Just-in-time permissions** are granted for the workflow and removed afterwards. **Watch the Lag column** before cutover (cutover = planned failover, so the secondary must catch up).

---

## 📋 **Prerequisites & Limitations**

- An Azure subscription + SQL Server **enabled by Azure Arc** with the **latest Azure extension for SQL Server**.
- Prepare the environment for the chosen method (network/certificates for the link, storage account and backups for LRS).
- Version note: the Learn unit says **SQL Server 2012+** (depends on the method). The Arc docs say migration is available by default **from SQL Server 2014**.
- **Failover cluster instances (FCI)** can't be migrated with either method.
- Limitations of **MI link** and **LRS** still apply (e.g., no FILESTREAM/FileTable, 30-day LRS limit).
- **Reverse migration** (MI → SQL Server) isn't available in the Arc experience. Use backup/restore or configure the link manually in SSMS. Whether it's possible depends on the MI **update policy**.
- Portal **monitoring** requires the instance to meet the Arc monitoring **licensing** requirements.

---

## ✅ **Summary**

- **Arc migration = portal-based, end-to-end**: assess → select target → migrate → monitor & cut over.
- **Weekly automatic assessments** with right-sized MI recommendations, plus **Copilot** help.
- Engine: **MI link** (best for minimal downtime and BC) or **LRS** (most compatible, older versions, all editions).

---

## 🎯 **Exam Tips / Traps**

- "Arc-enabled SQL Server, migrate to MI **from the Azure portal** with assessment + target sizing" → **SQL Server migration in Azure Arc**.
- "Which methods does the Arc migration use?" → **MI link** and **LRS** only (not DMS, not BACPAC).
- "Need a **readable** target during migration / **Business Critical** target / seconds of downtime" → choose **MI link**.
- "SQL Server **2012**, or **Web/Express** edition" → **LRS** (the link needs 2016+ and Enterprise/Standard/Developer).
- "Source is a **failover cluster instance**" → not supported by Arc migration.

> Source: https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-managed-instances/3-azure-arc · https://learn.microsoft.com/en-us/sql/sql-server/azure-arc/migrate-to-azure-sql-managed-instance · https://learn.microsoft.com/en-us/training/modules/migrate-sql-workloads-azure-managed-instances/5-use-link-feature
