# Alerts, monitoring and troubleshooting automated tasks

> **Exam skills:** Configure alerts and notifications on database tasks, Troubleshoot automated database tasks
> **Reading time:** ~5 min

## The big idea
An automated task that fails quietly is worse than no task. You need to **see** each run, **get told** when one fails, and **find out why**. It is like a smoke alarm: it does not stop the fire, but it wakes you up in time.

## Azure Monitor alerts and action groups
In Azure, alerts come from **Azure Monitor**. An alert rule has three parts:
1. **Scope** – the resource to watch (an Automation account, an elastic job agent, a database).
2. **Condition** – the signal and threshold. Example: "Failed elastic jobs greater than 0".
3. **Action group** – what to do when it fires.

An **action group** can:
- Send **email or SMS**.
- Call a **webhook** (a web address another system listens on).
- Start an **Automation runbook** (for example, to fix the problem itself).

You can filter an alert by **dimension**, such as one runbook name.

**Example:** A shop's nightly "PurgeOldOrders" runbook fails. An alert on the Automation account's job metric, filtered to that runbook and status Failed, emails the DBA team through an action group.

## Monitor Azure Automation runbooks
| Where | What you see |
|---|---|
| Automation account → **Jobs** | Each run and its status (completed, failed, running) |
| A job's **Errors** / **All logs** streams | The error text and output for that run |
| **Activity log** | **Who** started a runbook and when |
| **Log Analytics** | Job logs you can query with KQL, across many runs |

- To send job logs to Log Analytics, first turn on **Diagnostic settings** for the Automation account.

```kusto
AzureDiagnostics
| where Category == "JobLogs"
| where ResultType == "Failed"
```
KQL = Kusto Query Language.

## Monitor elastic jobs
| Where | What you see |
|---|---|
| Portal → elastic job agent → **Overview** | Job runs and status |
| PowerShell | `Get-AzSqlElasticJobExecution`, plus `Get-AzSqlElasticJobStepExecution` and `Get-AzSqlElasticJobTargetExecution` for **each step** and **each target database** |
| T-SQL in the job database | The `jobs.job_executions` view |

```sql
-- Failed runs, newest first
SELECT job_name, target_database_name, lifecycle, last_message, start_time
FROM jobs.job_executions
WHERE lifecycle = 'Failed'
ORDER BY start_time DESC;

-- Jobs running now
SELECT * FROM jobs.job_executions WHERE is_active = 1;
```

- Elastic jobs **retry** on short, temporary errors (such as a network blip).
- **Alerts:** the job agent works with **Azure Monitor**. Use the **Failed Elastic jobs** metric with an action group.
- History in `jobs.job_executions` is **deleted after 45 days**. You can purge it sooner with `jobs.sp_purge_jobhistory`.
- Give people who only need to watch jobs the **`jobs_reader`** role in the job database.

**Example:** An elastic job adds a column to 300 customer databases. It shows "Failed". `Get-AzSqlElasticJobTargetExecution` shows only 2 databases failed, because the job identity has no user in them. Add the user and rerun. The step is idempotent, so the 298 good databases are not harmed.

## SQL Server Agent jobs
Use Agent notifications and alerts (lesson 3) and job history (lesson 4). On **Managed Instance** there are no Agent alerts, so use **Azure Monitor alerts**.

## Troubleshooting checklist
| Symptom | Check |
|---|---|
| Runbook fails at sign-in | Managed identity is on and has the right **RBAC role** |
| Runbook says a cmdlet is not found | The module is **imported** (Az.Accounts before Az.Sql; SqlServer for `Invoke-Sqlcmd`) |
| Elastic job fails on some databases | Per-target results. Does the job identity have a user in each target? Is the target reachable (firewall, private endpoint approved)? |
| Logic App never fires on new rows | Polling interval; in Standard, **change tracking** on the table |

## Remember for the exam
- Azure alerts = **Azure Monitor alert rule + action group**.
- Action group can email, SMS, call a **webhook**, or start a **runbook**.
- Runbook status → Automation account **Jobs**. Who ran it → **activity log**.
- Runbook logs in Log Analytics need **Diagnostic settings** first.
- Elastic job alert → **Failed Elastic jobs** metric.
- Elastic job history: `jobs.job_executions`, kept **45 days**. Watchers get **`jobs_reader`**.
- Per-target failures → `Get-AzSqlElasticJobTargetExecution`.
- MI has no Agent alerts → **Azure Monitor**.

## Quick check
1. You want an email when any elastic job fails. What do you set up?
   <details><summary>Answer</summary>An Azure Monitor alert on the Failed Elastic jobs metric of the job agent, with an action group that sends email.</details>
2. You need to know who started a runbook yesterday. Where do you look?
   <details><summary>Answer</summary>The activity log of the Automation account.</details>
3. You want to query runbook job logs with KQL, but the AzureDiagnostics table is empty. What is missing?
   <details><summary>Answer</summary>Diagnostic settings on the Automation account, sending job logs to the Log Analytics workspace.</details>
