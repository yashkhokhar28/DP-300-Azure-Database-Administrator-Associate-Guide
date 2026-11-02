# SQL Server Agent jobs

> **Exam skills:** Manage schedules for regular maintenance jobs, Configure job alerts and notifications, Configure alerts and notifications on database tasks
> **Reading time:** ~8 min

## The big idea
Databases need regular care: backups, corruption checks, index and statistics upkeep. **SQL Server Agent** is the built-in scheduler that does this for you. It is like a night manager with a checklist. Each night the manager does the list. If something goes wrong, the manager phones the boss.

## Where SQL Server Agent exists
| Platform | SQL Server Agent? |
|---|---|
| **SQL Server on an Azure VM** | **Yes, full Agent.** You manage the service, its account, proxies and alerts. |
| **Azure SQL Managed Instance** | **Yes, built in and always running.** You cannot stop or start it. Its settings are **read-only**. Some features are missing (see lesson 4). |
| **Azure SQL Database** | **No.** It has no `msdb` database (where Agent stores jobs). Use **elastic jobs**, Azure Automation or Logic Apps instead (lesson 5). |

## Jobs, steps and schedules
- A **job** is a named task. Example: "NightlyBackup".
- A job has one or more **steps**. A step can be T-SQL, PowerShell, an SSIS package (SQL Server Integration Services, for moving data), or CmdExec (an operating-system command).
- A **schedule** says when a job runs: once, daily, weekly, at Agent start, and so on.
- Jobs and schedules have a **many-to-many** link. One schedule can drive many jobs. One job can have many schedules.
- Jobs, steps, schedules and history live in the **`msdb`** system database.

## Example schedule
A shop database backs up like this:

| When | What |
|---|---|
| Sunday 2 AM | Full backup |
| Monday–Saturday 1 AM | Differential backup |
| Every hour | Transaction log backup (lets you restore to a point in time) |
| Saturday midnight | Index maintenance |

## Maintenance plans
- The **Maintenance Plan Wizard** in SSMS (SQL Server Management Studio) builds maintenance tasks for you.
- It saves each plan as an **SSIS package** and creates a SQL Server Agent **job** to run it.
- The wizard creates a **separate schedule** for each plan.
- You choose who the job runs as: the **SQL Agent service account** or a **proxy account**.

Tasks you can add:
| Task | What it does |
|---|---|
| Check database integrity | Runs `DBCC CHECKDB` |
| Shrink database | Makes files smaller. **Avoid doing it often** — it fragments indexes. |
| Reorganize / Rebuild index | Fixes fragmentation (index pages out of order) |
| Update statistics | Helps the optimizer pick good query plans |
| History cleanup | Deletes old backup and job history in `msdb` |
| Back up database | Full, differential or log backup |
| Maintenance cleanup | Deletes old backup and report files from disk |
| Execute SQL Server Agent job | Runs another existing job |

Good habits:
- Run heavy tasks (index rebuild, CHECKDB) **at quiet times**.
- **Keep backups longer** than the gap between CHECKDB runs, as corruption may be found late.
- **Do not shrink** databases on a schedule.

## Proxy accounts
- A **proxy** lets a CmdExec, PowerShell or SSIS step run as a **different Windows account**, built on a **credential** (a stored account name and secret). **T-SQL steps never use a proxy.** Managed Instance does not support proxies.
- **Example:** A step must copy backups to a locked file share. A proxy with access to that share runs the step.

## Multi-server administration
On SQL Server, a **master server (MSX)** can push jobs to many **target servers (TSX)**. Managed Instance does not support this.

## Operators, notifications and alerts
| Object | What it is |
|---|---|
| **Operator** | A person or group that receives messages, usually by email |
| **Notification** | A message sent when a **job** succeeds, fails or completes |
| **Alert** | A rule that fires on an **error** or **condition** and tells an operator or runs a job |

### Operators
- An operator has a name and an **email address**.
- **Best practice:** use a **group email**, so nothing is missed when one person is away.
- **Example:** operator `DBATeam` with email `dba-support@shop.com`.

### Job notifications
- On the job's **Notifications** page, choose an operator and **when** to send: on **failure** (most common), **success** or **completion**.
- **Example:** email `DBATeam` when the nightly backup **fails**.

### Alerts
An Agent alert can fire on:
| Type | Example |
|---|---|
| **SQL error number** | Errors **823, 824, 825** (disk I/O errors that can mean corruption) |
| **Severity level** | All errors of severity **16 or higher** |
| **Performance condition** | Page Life Expectancy below 300 (memory pressure) |
| **WMI event** | An availability group failover |

When an alert fires it can:
1. Notify an operator (usually email).
2. **Run a job** to react. Example: on an 823 error, start a job that runs `DBCC CHECKDB`.

**Managed Instance:** SQL Agent **alerts are not supported**. Pager and NetSend are not supported either. Only **email notifications on job results** work. For anything else on MI, use **Azure Monitor alerts**.

## Database Mail
Agent sends email through **Database Mail**. Set it up like this:

**SQL Server (VM):**
1. Turn on Database Mail and create a **mail profile** (SMTP server, account).
2. In **SQL Server Agent Properties → Alert System**, tick **Enable mail profile** and choose the profile.
3. **Restart SQL Server Agent.**

**Managed Instance:**
- Agent properties are read-only, so you cannot pick a profile there.
- Instead, the Database Mail profile **must be named `AzureManagedInstance_dbmail_profile`**.
- The subnet's **NSG** (network security group, a firewall for the subnet) must allow **outbound** traffic to the SMTP port (25, 587 or 465).

```sql
EXEC msdb.dbo.sp_add_operator @name = N'DBATeam', @email_address = N'dba-support@shop.com';
EXEC msdb.dbo.sp_update_job  @job_name = N'NightlyBackup',
     @notify_level_email = 2, @notify_email_operator_name = N'DBATeam';  -- 2 = on failure
```
`@notify_level_email`: **1** = on success, **2** = on failure, **3** = on completion.

## Remember for the exam
- Agent exists on **VM** and **Managed Instance**. **Not on Azure SQL Database** → elastic jobs.
- On MI, Agent is **always on** and its settings are **read-only**.
- Jobs, schedules and history are stored in **`msdb`**. Maintenance plans are **SSIS packages**.
- Jobs and schedules are **many-to-many**.
- **Don't shrink** databases on a schedule.
- Operator = who. Notification = job result. Alert = error, severity, performance condition or WMI event.
- MI: **no Agent alerts, no proxies**. Mail profile must be **`AzureManagedInstance_dbmail_profile`**.
- SQL Server: enable the mail profile in Agent properties, then **restart Agent**.
- An alert can **run a job** to fix the problem.

## Quick check
1. On a Managed Instance, job emails never arrive. Agent properties cannot be edited. What do you check first?
   <details><summary>Answer</summary>That the Database Mail profile is named AzureManagedInstance_dbmail_profile, and that the NSG allows outbound SMTP.</details>
2. You want an email when error 824 appears on SQL Server on a VM. What do you create?
   <details><summary>Answer</summary>A SQL Server Agent alert on error number 824, with an operator to notify.</details>
3. A job step must copy files to a share the Agent service account cannot reach. What do you use?
   <details><summary>Answer</summary>A proxy account based on a credential that has access to the share (VM only, not MI).</details>
