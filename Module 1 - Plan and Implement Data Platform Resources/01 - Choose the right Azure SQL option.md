# Choose the right Azure SQL option

> **Exam skills:** Recommend a database offering based on specific requirements, Identify use cases for Azure Arc-enabled SQL services, Identify use cases for Azure SQL Database in Microsoft Fabric
> **Reading time:** ~10 min

## The big idea
Azure has three main homes for SQL Server data. **SQL Server on Azure VM** gives you full control. **Azure SQL Managed Instance** (MI) is almost full SQL Server, but Microsoft runs it. **Azure SQL Database** is a single cloud database with the least admin work. Two extras sit beside them: **Azure Arc** (manage SQL Server outside Azure) and **SQL database in Microsoft Fabric** (a database built for analytics).

Analogy: your own house (VM), a serviced flat (MI), a hotel room (SQL Database). The more service you get, the less you can change.

## Who manages what
| Layer | SQL Database (PaaS) | Managed Instance (PaaS) | SQL Server on VM (IaaS) |
|-------|--------------------|------------------------|------------------------|
| Hardware | Microsoft | Microsoft | Microsoft |
| OS and patching | Microsoft | Microsoft | **You** |
| SQL Server settings | Partial | Extensive | **Full** |
| Network and storage | No | Partial (own subnet) | **Full** |
| Backups and HA | Built in | Built in | You set them up |

**PaaS** (Platform as a Service) means Microsoft runs the server for you. **IaaS** (Infrastructure as a Service) means Azure gives you a virtual machine and you run everything inside it.

## SQL Server on Azure VM (IaaS)
It works like SQL Server on your own server. Pick it when you need:
- **Old SQL Server versions** that a vendor still requires.
- **SSIS, SSAS, SSRS** (Integration, Analysis, Reporting Services) on the same machine.
- **OS access** to install agents, scripts or other apps.
- Features PaaS does not have: **FILESTREAM, FileTable, PolyBase, cross-instance transactions**.
- A plain **lift-and-shift** (move as-is) with no changes.

Marketplace images today cover SQL Server **2016 to 2025**. 2012/2014 images are gone; install those by hand.

**Example:** A factory runs an old SQL Server 2016 app plus SSRS reports on one box. The vendor will not support anything newer. Choose a SQL Server VM.

## Azure SQL Managed Instance (PaaS)
A full SQL Server **instance** run by Microsoft. It is close to 100% compatible with on-premises SQL Server. It has things SQL Database lacks:
- **SQL Agent** jobs, **linked servers**, **CLR**, **Service Broker**.
- **Cross-database queries** inside the instance.
- System databases (msdb and others).
- **Restore native .bak backups** from Azure Blob Storage.
- Many databases per instance (up to 100; 500 on Next-gen General Purpose).
- Lives in your own **virtual network** (private by default).

**Example:** A clothing retailer's catalog database uses SQL Agent jobs and cross-database joins. They want no code changes and no patching. Choose MI.

## Azure SQL Database (PaaS)
One database (or a pool of them) with the least admin work. Best for **new cloud apps** and **unknown or spiky load**. Features you only get here:
- **Hyperscale** up to 128 TB per database.
- **Serverless** auto-scale and auto-pause.
- **Elastic jobs** and **elastic query** across databases.
- **Automatic tuning** that can create and drop indexes.

**Example:** A startup builds a new booking app. It does not know how busy it will be. Choose SQL Database, start small, scale later.

## Compare at a glance
| Need | Best choice |
|------|-------------|
| Lift and shift, no changes, OS access | SQL Server on VM |
| Old SQL version, SSIS/SSRS/SSAS on same box | SQL Server on VM |
| FILESTREAM, FileTable, PolyBase | SQL Server on VM |
| SQL Agent, linked servers, CLR, cross-db queries, few code changes | Managed Instance |
| Restore native .bak files from on-premises | Managed Instance (or VM) |
| New cloud-native app, least admin | SQL Database |
| Many small databases with uneven load | SQL Database elastic pool |
| Database over 4 TB, few app changes | SQL Database Hyperscale |
| Big data warehouse, huge scans (MPP) | Azure Synapse Analytics |

Migration effort for an old app: SQL Database = high, MI = low, VM = very low.

**Azure Synapse Analytics** is for big analytics only. It is bad for many small reads and writes.

## Azure Arc-enabled SQL Server
**Azure Arc** puts SQL Servers that run outside Azure (your data centre, other clouds, the edge) into the Azure portal. The server stays where it is.

Use cases:
- **One inventory** of all SQL Servers everywhere.
- **Best-practice assessments** and **security scans** with Microsoft Defender for Cloud.
- **Governance** with Azure Policy and Microsoft Purview.
- **Patching** through Azure Update Manager or Arc automatic updates.
- **Pay-as-you-go licensing** billed through Azure.
- **Extended Security Updates (ESU)** for SQL versions past end of support.
- **Migration assessment and migration to MI** from the portal (lesson 7).
- Automated backups (preview) and Microsoft Entra authentication.

**Example:** A company has SQL Servers in 20 remote factories. It wants one place to see versions, scan for weak settings and patch them. It does not want to move them. Arc-enable them.

## SQL database in Microsoft Fabric
The exam says "Azure SQL Database in Microsoft Fabric". The docs say "SQL database in Fabric". Same thing.

It is an **OLTP** (day-to-day transactions) database that lives **inside a Fabric workspace**. It is SaaS, not an Azure resource. It uses the same engine as Azure SQL Database. It is **generally available since Nov 2025**.

Its key feature: every table is **automatically mirrored** to **OneLake** in near real time, as Delta (Parquet) files. A read-only **SQL analytics endpoint** is created for you. Power BI and Spark can read the data with **no ETL** (no copy jobs). Mirroring cannot be turned off.

Main limits compared with Azure SQL Database:
- **Microsoft Entra authentication only.** No SQL logins.
- Max **4 TB**, auto-scale up to **32 vCores**.
- **No** geo-replication, failover groups, geo-restore or **LTR** (long-term backup retention).
- Backups: point-in-time restore 7 days by default, up to 35.
- **No** Always Encrypted, ledger, In-Memory OLTP or CDC.
- **No** elastic pools (a Fabric **capacity** plays that role) and **no** SQL Agent or elastic jobs. Use **Data Factory pipelines** to schedule work.
- **RLS, dynamic data masking and object permissions are not copied** to the OneLake mirror.
- Deploy with the Fabric portal, REST API or Fabric CLI. No ARM, Bicep, Az PowerShell or Azure CLI.

Use it for:
- **Translytical apps** (transactions plus near real-time analytics on the same data).
- **Reverse ETL** (push lakehouse data back into an app database).
- An **operational data store** or **AI/RAG apps** close to Fabric data.

If you only need an existing Azure SQL Database copied into OneLake, use **Fabric mirroring** instead. On MI, mirroring needs the **SQL Server 2025 or Always-up-to-date** update policy.

**Example:** A shop already uses Fabric and Power BI. It wants new orders to show up in dashboards within seconds, with no ETL jobs. Choose SQL database in Fabric.

## Remember for the exam
- "Minimal code changes + SQL Agent/linked servers/CLR" → **Managed Instance**.
- "Need OS access, FILESTREAM or an old version" → **SQL Server on VM**.
- "New cloud app, least admin, unpredictable load" → **SQL Database** (serverless).
- Elastic jobs, elastic query and Hyperscale are **SQL Database only**.
- "Manage on-premises and multi-cloud SQL Servers from Azure" → **Azure Arc**.
- "OLTP data in Power BI in near real time with no ETL" → **SQL database in Fabric** (or Fabric mirroring).
- Fabric SQL database: **Entra-only**, no geo-DR, no LTR, no Always Encrypted. If the question needs any of these, pick Azure SQL Database.
- Don't confuse the Fabric SQL database (read/write) with its **SQL analytics endpoint** (read-only).

## Quick check
1. An app uses SQL Agent jobs and linked servers. The team wants Microsoft to handle patching. Which option?
   <details><summary>Answer</summary>Azure SQL Managed Instance. It supports Agent and linked servers and is PaaS.</details>
2. A team must use SQL authentication and geo-restore. Can they use SQL database in Fabric?
   <details><summary>Answer</summary>No. Fabric SQL database is Entra-only and has no geo-restore. Use Azure SQL Database.</details>
3. You have 300 on-premises SQL Servers and want one inventory and security view in Azure without moving them. What do you use?
   <details><summary>Answer</summary>Azure Arc-enabled SQL Server. It brings outside servers into Azure management.</details>
