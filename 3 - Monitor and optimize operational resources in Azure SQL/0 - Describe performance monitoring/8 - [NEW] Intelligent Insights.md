### ✅ **Azure SQL: Intelligent Insights — Explained**

---

## 🧠 **What Is Intelligent Insights?**

- **Built-in AI** feature that **continuously monitors database usage** and **detects disruptive events** that cause poor performance.
- Applies to **Azure SQL Database** (single + pooled) **and Azure SQL Managed Instance**.
- When it detects a problem it writes a **diagnostic log entry called `SQLInsights`** containing:
  - **Root cause analysis** (human-readable)
  - **Impacted queries** (query hashes) / error codes
  - **Performance improvement recommendations** (where possible)
- ⚠️ **Status (Oct 2026): still a _preview_ feature** and **not available in West Europe, North Europe, West US 1 and East US 1**.
- ⚠️ `SQLInsights` (the log category) is **unrelated to "Azure Monitor SQL Insights"**, which was **retired on 31 Dec 2024** (replacement: **database watcher**).

---

## ⚙️ **How It Works**

| Concept | Detail |
| --- | --- |
| **Baseline** | Compares the workload of the **last hour** against a **7-day baseline** workload |
| **Workload** | The most significant queries (most repeated + largest) — unique per database |
| **Absolute thresholds** | Independently of the baseline, also watches excessive waits, critical exceptions, parameterization issues |
| **Lifecycle tracking** | Each issue tracked from detection → verifying → complete (`status_s` = **Active**, **Verifying**, **Complete**) |
| **Latency** | Usually needs **~1 hour** to finish root cause analysis → for a problem happening _right now_, use **Query Store** directly |

**4 detection metrics:**

1. **Query duration** (compile or execution time increase) — primary model
2. **Timeout requests** — primary model
3. **Excessive wait times** (from Query Store wait stats)
4. **Errored-out requests** (incl. critical exceptions)

---

## 🔍 **The 15 Detectable Performance Patterns**

| Pattern | What it means / typical fix |
| --- | --- |
| **Reaching resource limits** | CPU/DTU, worker or session limits hit → tune queries, or scale up |
| **Workload increase** | More/accumulating workload ("pile-up") vs baseline → tune top contributor, distribute, scale |
| **Memory pressure** | Workers waiting on memory grants → tune queries, add indexes, scale up |
| **Locking** | Excessive locking/blocking → shorter transactions, smaller batches, indexes |
| **Increased MAXDOP** | Plan more parallel than it should be (CXP waits) → set MAXDOP / simplify query |
| **Pagelatch contention** | Threads fighting over same in-memory pages (e.g., sequential key inserts) → non-sequential key, hash partitioning |
| **Missing index** | Index would help identified queries → create index / enable **automatic tuning** |
| **New query** | New poorly performing query (up to 2 top-CPU new queries reported) |
| **Increased wait statistic** | Regression with a wait type but no other category matched → manual troubleshooting |
| **TempDB contention** | Many concurrent queries creating/dropping small temp tables → memory-optimized tables |
| **Elastic pool DTU shortage** | Not enough **eDTUs** in the pool → tune top consumers / raise pool tier (**SQL DB only**) |
| **Plan regression** | New/old plan regression or alternating plans → force good plan / **FORCE_LAST_GOOD_PLAN** |
| **Database-scoped configuration value change** | Recent DSC change (MAXDOP, LEGACY_CARDINALITY_ESTIMATION, PARAMETER_SNIFFING...) hurt perf → revert |
| **Slow client** | App/network can't consume results fast enough (ASYNC_NETWORK_IO-style) → fix client side |
| **Pricing tier downgrade** | Tier was lowered and performance dropped → scale back up or reduce load |

> 💡 **Elastic pool DTU shortage** is the **only pattern not available for Managed Instance** (MI is vCore only).

---

## 📤 **Enable It — Diagnostic Settings**

Intelligent Insights output is **not stored by default** — streaming export of diagnostic telemetry **isn't enabled by default**. You must create a **diagnostic setting** on the **database** and tick **`SQLInsights`**.

| Destination | Use for |
| --- | --- |
| **Log Analytics workspace** | KQL queries, log alerts, workbooks (table: **`AzureDiagnostics`**, `Category == "SQLInsights"`) |
| **Event Hubs** | Stream to SIEM / third-party / custom alerting |
| **Azure Storage** | Cheap long-term archive, custom reporting |

- **Managed Instance:** configure it on **each instance database** (the MI resource itself only emits `ResourceUsageStats`); you **can't** configure diagnostic settings on MI **system databases**.
- **Elastic pool:** pool resource only emits Basic metrics → configure **SQLInsights on each pooled database**.

```powershell
# Az.Monitor (current cmdlets) – send SQLInsights to Log Analytics
$dbId  = (Get-AzSqlDatabase -ResourceGroupName "rg-sql" -ServerName "sql-prod-01" -DatabaseName "SalesDb").ResourceId
$wsId  = (Get-AzOperationalInsightsWorkspace -ResourceGroupName "rg-mon" -Name "law-sql").ResourceId
$log   = New-AzDiagnosticSettingLogSettingsObject -Enabled $true -Category SQLInsights

New-AzDiagnosticSetting -Name "diag-sqlinsights" -ResourceId $dbId -WorkspaceId $wsId -Log $log
```

```bash
# Azure CLI
az monitor diagnostic-settings create \
  --name diag-sqlinsights \
  --resource "/subscriptions/<sub>/resourceGroups/rg-sql/providers/Microsoft.Sql/servers/sql-prod-01/databases/SalesDb" \
  --workspace "/subscriptions/<sub>/resourceGroups/rg-mon/providers/Microsoft.OperationalInsights/workspaces/law-sql" \
  --logs '[{"category":"SQLInsights","enabled":true}]'
```

> Older docs still show `Set-AzDiagnosticSetting`; current Az.Monitor uses **`New-AzDiagnosticSetting`** + `New-AzDiagnosticSettingLogSettingsObject`.

---

## 🔎 **Query the Insights (KQL)**

```kusto
// Active Intelligent Insights in the last 24 hours
AzureDiagnostics
| where Category == "SQLInsights" and status_s == "Active"
| where TimeGenerated > ago(24h)
| project TimeGenerated, databaseName_s, elasticPoolName_s, issueId_d, rootCauseAnalysis_s, detections_s
```

Key log fields: `issueId_d` (tracks one issue end-to-end), `status_s`, `detections_s` (**impact 1–3**: 1 low, 2 moderate, 3 high), `impact_s` (query hash + metric such as `DurationIncreaseSeconds`, `WaitingSeconds`, `CriticalErrorCount`), `rootCauseAnalysis_s`.

> Use this as a **log search alert rule** to get notified (e.g., `rootCauseAnalysis_s contains "hitting its CPU limits"`).

---

## 🔗 **How It Relates to Other Tools**

| Tool | Relationship |
| --- | --- |
| **Query Performance Insight (QPI)** | Portal view of top CPU/IO/duration queries from **Query Store** (**SQL DB only**). QPI = *"which queries are expensive"*; Intelligent Insights = *"something regressed vs baseline, and here's why"* |
| **Query Store** | Data source for wait stats / plans; go here for issues **< 1 hour old** or to force a plan |
| **Automatic tuning** | Docs recommend enabling it alongside — **acts** on findings (**FORCE_LAST_GOOD_PLAN** on SQL DB + MI; **CREATE/DROP INDEX on SQL DB only**) |
| **Database watcher (preview)** | Newer managed monitoring (DMV data → Azure Data Explorer / Fabric Real-Time Analytics, dashboards, alerts). Replacement for retired SQL Insights |
| **Azure SQL Analytics (preview)** | Legacy Log Analytics "solution" that visualized SQLInsights. **Not formally retired**, but docs moved to the **archive ("previous versions")** and say it's **no longer in active development** → don't build new monitoring on it |

---

## ✅ **Summary**

| Topic | Key Point |
| --- | --- |
| What | AI-based detection of performance degradation + root cause analysis |
| Baseline | Last hour vs **7-day** baseline |
| Output | **`SQLInsights`** resource log (JSON) |
| Enable | **Diagnostic settings** on each database → Log Analytics / Event Hubs / Storage |
| Applies to | **SQL Database + SQL Managed Instance** (preview) |
| Patterns | **15**; elastic pool DTU shortage = SQL DB only |
| Act on it | Automatic tuning, Query Store plan forcing, scale up, index changes |

---

## 🎯 **Exam Tips / Traps**

- **"Need root cause analysis of a performance regression streamed to Event Hubs/Log Analytics"** → enable **`SQLInsights`** in **Diagnostic settings**.
- Baseline is **7 days**; analysis takes **~1 hour** → for real-time issues use **Query Store / DMVs**.
- Insights are **not collected/retained by default** — no diagnostic setting = no insights to query.
- **Archive cheaply / long-term** → **Storage account**; **third-party SIEM** → **Event Hubs**; **KQL + alerts** → **Log Analytics**.
- **Elastic pool DTU shortage** won't appear on **Managed Instance**.
- MI: set diagnostics on **instance databases**, not system DBs.
- Don't confuse **`SQLInsights` log** (Intelligent Insights, still exists) with **SQL Insights (preview)** (Azure Monitor product, **retired 31 Dec 2024**).
- **Plan regression** detected → the fix is **plan forcing / automatic tuning FORCE_LAST_GOOD_PLAN**.
- **QPI is SQL Database only**; Intelligent Insights covers **SQL DB and MI**.

---

> Source: https://learn.microsoft.com/en-us/azure/azure-sql/database/intelligent-insights-overview, https://learn.microsoft.com/en-us/azure/azure-sql/database/intelligent-insights-troubleshoot-performance, https://learn.microsoft.com/en-us/azure/azure-sql/database/intelligent-insights-use-diagnostics-log, https://learn.microsoft.com/en-us/azure/azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure, https://learn.microsoft.com/en-us/azure/azure-sql/database/monitoring-sql-database-azure-monitor-reference, https://learn.microsoft.com/en-us/previous-versions/azure/azure-monitor/insights/azure-sql, https://learn.microsoft.com/en-us/azure/azure-sql/database-watcher-overview, https://learn.microsoft.com/en-us/powershell/module/az.monitor/new-azdiagnosticsetting, https://learn.microsoft.com/en-us/cli/azure/monitor/diagnostic-settings
