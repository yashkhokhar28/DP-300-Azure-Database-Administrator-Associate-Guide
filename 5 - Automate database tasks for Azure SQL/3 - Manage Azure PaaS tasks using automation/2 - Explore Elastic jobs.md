## ✅ **Key Concept: Elastic Jobs for Azure SQL Database**

Elastic Jobs let you:

- **Schedule or run on-demand T-SQL scripts**
- **Target multiple databases or servers** in one job
- Automate repetitive maintenance or deployment tasks
- Similar to SQL Server Agent jobs but **limited to T-SQL only**
- **Generally available** since April 2024; works with **all tiers** of Azure SQL Database (targets can span servers, pools, regions and subscriptions). 🔄

---

## 🔨 **Main Components of Elastic Jobs**

| Component             | Purpose                                                                |
| --------------------- | ---------------------------------------------------------------------- |
| **Elastic Job Agent** | Azure resource that manages job execution                              |
| **Job Database**      | SQL Database that stores job metadata (Recommended: S1 tier or higher) |
| **Job agent tier** 🔄 | Concurrency capacity: **JA100** (default, 100 concurrent targets), **JA200**, **JA400**, **JA800** |
| **Target Group**      | A group of databases, servers, or elastic pools where jobs run         |
| **Job / Job Step**    | The T-SQL scripts you want to run                                      |

---

### 🎯 **Example Scenario**

If you need to update a table structure or gather usage metrics across **30 different Azure SQL Databases**, Elastic Jobs allow you to run a single job against all of them.

---

## ⚙️ **Typical Setup Process**

1. **Create a Job Agent:**

   - Through Azure Portal, PowerShell, or REST API.
   - Linked to a **Job Database**, where job configurations are stored.
   - Job database should be a new, empty database; **Hyperscale** and **auto-pause serverless** databases aren't supported as the job database. 🔄

2. **Create Target Groups:**

   - A target group can be:

     - All databases on a server
     - A single database
     - An elastic pool

   - You must set up **credentials** (with least-privilege access) to allow the job agent to connect.
   - **Recommended auth: Microsoft Entra ID with a user-assigned managed identity (UMI)** on the job agent (system-assigned MI isn't supported). **Database-scoped credentials** (in the job database, plus a `master` credential for server/pool enumeration) are the alternative — one method per agent. 🔄
   - Target servers with public access denied → use **elastic job private endpoints** (service-managed; approve on each target server). 🔄
   - T-SQL alternative: `jobs.sp_add_target_group` / `jobs.sp_add_target_group_member` in the job database. 🔄

   Example in PowerShell:

   ```powershell
   $serverGroup = $jobAgent | New-AzSqlElasticJobTargetGroup -Name 'MyServerGroup'
   $serverGroup | Add-AzSqlElasticJobTarget -ServerName $targetServerName -RefreshCredentialName $masterCred.CredentialName
   ```

3. **Create the Job and Job Steps:**

   - Each step runs a T-SQL command.
   - Example T-SQL step to create a table if it doesn't exist:

   ```sql
   IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id('MyTable'))
   CREATE TABLE [dbo].[MyTable]([Id] [int] NOT NULL);
   ```

   - Jobs should be **idempotent** (safe to run multiple times without side effects).

4. **Run the Job:**

   - On-demand (`RunOnce`) or on a schedule.

   Example to start a job:

   ```powershell
   $jobExecution = $job | Start-AzSqlElasticJob
   ```

---

## 🔍 **Use Case Examples**

| Use Case                       | Example                                                             |
| ------------------------------ | ------------------------------------------------------------------- |
| Scheduled maintenance tasks    | Update statistics across many databases nightly                     |
| Schema deployment              | Add new tables or columns across dev/test/prod                      |
| Cross-database data collection | Pull metrics or usage data from all databases                       |
| Bulk data load                 | Load data from Azure Blob Storage                                   |
| Off-peak processing            | Run jobs overnight to avoid peak hour impact                        |
| Data aggregation               | Combine telemetry from multiple databases into a reporting database |

---

## ⚖️ **Elastic Jobs vs SQL Server Agent**

| Feature             | Elastic Jobs (Azure SQL Database) | SQL Server Agent (Managed Instance/SQL Server) |
| ------------------- | --------------------------------- | ---------------------------------------------- |
| Platform Support    | Azure SQL Database only           | SQL Managed Instance & SQL Server              |
| Languages Supported | **T-SQL only**                    | T-SQL, PowerShell, SSIS, CmdExec, etc.         |
| Scope               | Multiple databases, elastic pools | Single server (instance-wide)                  |
| Administration      | Azure Portal, PowerShell, T-SQL, REST API (no `az sql` elastic-job commands) 🔄 | SSMS, T-SQL                                    |
| Setup Complexity    | Medium                            | Low (already built-in)                         |

---
