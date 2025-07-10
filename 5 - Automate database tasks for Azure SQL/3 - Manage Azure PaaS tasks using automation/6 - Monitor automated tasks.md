## ✅ **Monitoring Runbooks (Azure Automation)**

### 🔍 **What are Runbooks?**

Runbooks are PowerShell or Python scripts that automate tasks. Monitoring ensures they run successfully and restart correctly if interrupted.

### ✔️ **How to Monitor Runbooks**

| Method                                       | Description                                                  |
| -------------------------------------------- | ------------------------------------------------------------ |
| **Azure Portal → Automation Account → Jobs** | View the status of each runbook execution                    |
| **Database, Storage, or Shared Files**       | Optionally store runbook progress for restarts               |
| **Built-in Logging**                         | Shows whether a runbook completed, failed, or is in progress |

---

### 🚨 **Create Alerts for Runbooks**

You can create **Azure Monitor alerts** to notify you of issues.

#### Example Alert:

- Signal: **Total Jobs**
- Condition: Greater than **10**
- Action: Send an email or run another runbook.

⚙️ **Optional: Filter Alerts by Dimension**
Example: Only alert on a specific runbook or failure type.

---

### 🔔 **Action Groups**

When an alert is triggered, it can:

- Send an **email/SMS**
- Call a **webhook**
- Start another **Automation runbook**

---

### 📑 **Activity Logs**

The **Activity Log** shows:

- Who started a runbook
- When it ran
- Runbook status

Example PowerShell to get who ran a runbook in the last day:

```powershell
$rgName = 'MyResourceGroup'
$accountName = 'MyAutomationAccount'
$runbookName = 'MyRunbook'
$startTime = (Get-Date).AddDays(-1)

# Query the logs and extract runbook details
(Get-AzLog -ResourceGroupName $rgName -StartTime $startTime) |
Where-Object { $_.Authorization.Action -eq 'Microsoft.Automation/automationAccounts/jobs/write' }
```

---

### 📊 **Log Analytics Integration**

Send runbook job results to **Log Analytics Workspace** to:

- Write advanced queries (Kusto Query Language)
- Visualize job status
- Correlate across multiple automation jobs

Example Kusto query to list completed jobs:

```kusto
AzureDiagnostics
| where Category == "JobLogs"
| where ResultType == "Completed"
```

**Important:** You must first enable **Diagnostic Settings** for the Automation Account.

---

## ✅ **Monitoring Elastic Jobs**

Elastic Jobs automate running **T-SQL scripts across multiple Azure SQL databases**.

### ✔️ **How to Monitor Elastic Jobs**

| Tool                                        | What You Can See                                       |
| ------------------------------------------- | ------------------------------------------------------ |
| Azure Portal → Elastic Job Agent → Overview | Job history & status                                   |
| PowerShell                                  | Execution details using `Get-AzSqlElasticJobExecution` |
| T-SQL                                       | Query job execution history from the job database      |

### Example PowerShell:

```powershell
# Get the last 5 job executions
$jobAgent | Get-AzSqlElasticJobExecution -Count 5
```

### Example T-SQL:

```sql
-- View all job executions
SELECT * FROM jobs.job_executions ORDER BY start_time DESC;

-- View executions for a specific job
SELECT * FROM jobs.job_executions WHERE job_name = 'MyJob' ORDER BY start_time DESC;

-- View currently running jobs
SELECT * FROM jobs.job_executions WHERE is_active = 1 ORDER BY start_time DESC;
```

---

### 🔄 **Automatic Retry**

Elastic Jobs automatically retry failed executions to handle **transient failures** like network glitches.

---

## 🛡️ **Key Takeaways**

- Monitor all automated tasks to detect issues early.
- Use **alerts** to notify admins of failures.
- Store progress info in logs or databases to restart interrupted jobs properly.
- Use **Log Analytics** for advanced monitoring and querying.
- Elastic Jobs and Runbooks have separate monitoring methods but both integrate into Azure Monitor.

---
