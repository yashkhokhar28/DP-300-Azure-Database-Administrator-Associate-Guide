# Automate tasks in Azure

> **Exam skills:** Create and configure elastic jobs, Create and configure database tasks by using automation
> **Reading time:** ~9 min

## The big idea
Azure SQL Database has **no SQL Server Agent**. So Azure gives you three other tools. **Elastic jobs** run T-SQL on many databases. **Azure Automation** runs PowerShell or Python scripts on a schedule. **Logic Apps** run low-code workflows when something happens. Think of a head office: one clerk sends the same instruction to every shop (elastic jobs), one IT person runs scripts (Automation), and one form starts a chain of tasks when an order arrives (Logic Apps).

## Elastic jobs
An **elastic job** runs **T-SQL** on a schedule or on demand, across **many databases** at once. It is the Azure SQL Database version of an Agent job.
- **Generally available since April 2024.** Works with **all tiers** of Azure SQL Database.
- Targets can span servers, pools, regions and subscriptions. **T-SQL only.**

**Example:** A SaaS app has one database per customer, 300 databases in total. A new column must be added to every one. One elastic job does it in a single run.

### Parts of an elastic job
| Part | What it is |
|---|---|
| **Elastic job agent** | The Azure resource that runs the jobs |
| **Job database** | An Azure SQL database that stores the job definitions and history. Use a **new, empty** database. **S1 or higher** is recommended. **Hyperscale** and **auto-pause serverless** are **not** supported for it. |
| **Job agent tier** | How many targets run at the same time: **JA100** (default, 100), JA200, JA400, JA800 |
| **Target group** | Where the job runs: a single database, **all databases on a server**, or an **elastic pool** |
| **Job and job steps** | The T-SQL to run |

### How to set it up
1. **Create the job agent** (portal, PowerShell or REST API) and link it to the job database. Note: there are **no `az sql` CLI commands** for elastic jobs.
2. **Set up authentication** to the targets. Choose **one** method per agent:
   - **Recommended: Microsoft Entra ID with a user-assigned managed identity (UMI)** on the job agent. A **system-assigned** managed identity is **not supported**.
   - Or **database-scoped credentials** in the job database. Server and pool targets also need a credential for `master`, so the agent can list the databases.
   - Give the identity only the rights it needs (least privilege).
3. **Create target groups**, then the **job and steps** (T-SQL below, run in the job database).
4. **Run it** once (`jobs.sp_start_job` or `Start-AzSqlElasticJob`) or on a schedule.

Targets with public access off → **elastic job private endpoints**, which you **approve** on each target server.

```sql
-- In the job database
EXEC jobs.sp_add_target_group N'AllCustomers';
EXEC jobs.sp_add_target_group_member N'AllCustomers',
     @target_type = N'SqlServer', @server_name = N'cust-sql.database.windows.net';
EXEC jobs.sp_add_job @job_name = N'AddLoyaltyColumn';
EXEC jobs.sp_add_jobstep @job_name = N'AddLoyaltyColumn', @target_group_name = N'AllCustomers',
     @command = N'IF COL_LENGTH(''dbo.Customer'',''Loyalty'') IS NULL
                  ALTER TABLE dbo.Customer ADD Loyalty int NULL;';
EXEC jobs.sp_start_job N'AddLoyaltyColumn';
```

### Make steps idempotent
Elastic jobs **retry** failed steps automatically. So write steps that are **idempotent** (safe to run twice). The `IF ... IS NULL` check above means a second run does nothing harmful.

## Azure Automation
**Azure Automation** is a cloud service that runs scripts for you. It works across **Azure and on-premises** machines.

**Example:** Every day at 6 AM a runbook starts a VM, runs a SQL backup script, then stops the VM at 7 AM to save money.

### Parts of an Automation account
| Part | What it is |
|---|---|
| **Automation account** | The container for everything below |
| **Runbook** | The script: **PowerShell**, **Python**, or a graphical workflow |
| **Modules** | PowerShell modules the runbook uses, such as **Az.Accounts** and **Az.Sql** |
| **Credentials** | Saved user names and passwords, read in a runbook with `Get-AutomationPSCredential` |
| **Managed identity** | How the runbook signs in to Azure: `Connect-AzAccount -Identity`. **Run As accounts were retired on 30 Sep 2023.** |
| **Hybrid Runbook Worker** | Runs runbooks on your own machines. Now **extension-based only**; the old agent-based worker was retired on 31 Aug 2024. |

### Build a runbook (main steps)
1. Create an **Automation account**.
2. Import modules: **Az.Accounts first**, then **Az.Sql**. `Invoke-Sqlcmd` needs the **SqlServer** module. PowerShell 7.4+ runtime environments already include a default Az version.
3. (Optional) Add a **credential**, for example a SQL login called `SQLUser`.
4. Create the runbook. Type **PowerShell**. Runtime **PowerShell 7.4** (or 7.6) is recommended. 7.1 and 7.2 are out of support. 5.1 is still available. Python runbooks use **3.10**.
5. Test in the **Test pane**, publish, link a **schedule**, and give the managed identity the right **RBAC roles**.

```powershell
Connect-AzAccount -Identity                       # sign in with the managed identity
$cred = Get-AutomationPSCredential -Name "SQLUser"
Invoke-Sqlcmd -ServerInstance "shop-sql.database.windows.net" -Database "shopdb" `
  -Username $cred.UserName -Password $cred.GetNetworkCredential().Password `
  -Query "EXEC dbo.PurgeOldOrders;"
```

Things to know:
- If a runbook is stopped part-way, it **restarts from the beginning**. Write it to be restart-safe. It can store its progress in a database or storage.

**Tags** (`key:value` labels such as `Environment:Production`) help runbooks pick resources. Tags are plain text, so never store secrets in them.

## Logic Apps
**Azure Logic Apps** is a serverless service for **workflows**. You draw them in a designer with little or no code. A workflow starts with a **trigger** and then runs **actions**.

**Example:** A new row is added to the `Orders` table. The Logic App reads the row, emails the sales team through Office 365, and posts in Microsoft Teams.

### The SQL Server connector
- It can run queries, insert/update/delete rows, run **stored procedures**, and start a workflow on table events.
- It works with **SQL Server** (on-premises or VM), **Azure SQL Database** and **Azure SQL Managed Instance**.
- Sign-in: SQL auth, Entra integrated, service principal, or **managed identity** (recommended).
- The **"When an item is created"** trigger **polls** the table on a timer and reads **one row** per event.
- In **Standard** workflows, built-in SQL triggers (such as **When a row is inserted**) need **change tracking** turned on for the table.
- Managed Instance without a data gateway → use the MI **public endpoint on port 3342**.

### Plan types
- **Consumption** (multi-tenant): pay per run, managed connectors.
- **Standard** (single-tenant): isolated, built-in SQL connector, VNet integration and private endpoints.
- Note: **Integration Service Environment (ISE) retired 31 Aug 2024.** Move to Standard.

## Compare at a glance: which tool when?
| Need | Pick |
|---|---|
| Run **T-SQL on many Azure SQL databases**, scheduled | **Elastic jobs** |
| Schedule T-SQL in Azure SQL Database (no Agent) | **Elastic jobs** |
| Schedule T-SQL, SSIS or other steps on **VM or Managed Instance** | **SQL Server Agent** |
| Run **PowerShell or Python** on a schedule; manage Azure resources; hybrid with on-premises | **Azure Automation** |
| **Event-driven**, low-code flow across services (email, Teams, SharePoint, storage) | **Logic Apps** |
| Small event-driven code (serverless) | **Azure Functions** |

**Example:** Every night, scale a database down to S0 and back up to S3 at 8 AM. That is a change to an Azure resource, not T-SQL inside the database, so use an **Azure Automation runbook**.

## Remember for the exam
- **Azure SQL Database has no Agent** → elastic jobs, Automation, Logic Apps.
- Elastic jobs = **T-SQL only**, many targets, **GA since April 2024**, all tiers.
- Job database: new and empty, **S1+**; **not Hyperscale**, **not auto-pause serverless**.
- Elastic job auth: **user-assigned** managed identity (recommended). **System-assigned is not supported.** Or database-scoped credentials plus a `master` credential.
- Steps must be **idempotent** (jobs retry).
- Runbooks sign in with **managed identity** (Run As retired). Import **Az.Accounts before Az.Sql**.
- Logic Apps Standard SQL triggers need **change tracking**. MI public endpoint = **port 3342**. **ISE is retired.**

## Quick check
1. You must add an index to 50 Azure SQL databases every week. Which tool?
   <details><summary>Answer</summary>Elastic jobs. They run T-SQL on many Azure SQL databases on a schedule.</details>
2. You create an elastic job agent and turn on its system-assigned managed identity for target access. Will it work?
   <details><summary>Answer</summary>No. Elastic jobs need a user-assigned managed identity (or database-scoped credentials).</details>
3. When a row is inserted into an Orders table, you must email the sales team, with no code. Which tool?
   <details><summary>Answer</summary>Logic Apps, with a SQL trigger and an Office 365 email action.</details>
