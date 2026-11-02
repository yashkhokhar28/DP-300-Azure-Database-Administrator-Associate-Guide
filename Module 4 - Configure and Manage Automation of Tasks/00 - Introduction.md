# Module 4: Configure and Manage Automation of Tasks

**Exam weight:** 15–20% · **Reading time:** ~50 min · **Files:** 7

## What you will learn
You will create Azure SQL resources with code, and fix deployments that fail. You will schedule jobs with SQL Server Agent. You will learn what replaces Agent in Azure SQL Database. You will set up alerts when a task fails.

Think of a pizza chain. It builds every new shop from one **blueprint** (deployment automation). Each night a manager with a checklist cleans and counts the cash (SQL Server Agent). Shops with no night manager get the checklist run by head office (elastic jobs, Azure Automation, Logic Apps). If a task fails, someone's phone rings (alerts).

## Lessons in this module
| # | Lesson | What it covers | Time |
|---|--------|----------------|------|
| 1 | [Automate deployments](<01 - Automate deployments.md>) | Declarative vs imperative, ARM, Bicep, PowerShell, Azure CLI | ~8 min |
| 2 | [Monitor and troubleshoot deployments](<02 - Monitor and troubleshoot deployments.md>) | What-if, deployment history, activity log, common errors, rollback | ~8 min |
| 3 | [SQL Server Agent jobs](<03 - SQL Server Agent jobs.md>) | Jobs, schedules, maintenance plans, operators, alerts, Database Mail | ~8 min |
| 4 | [Troubleshoot SQL Server Agent jobs](<04 - Troubleshoot SQL Server Agent jobs.md>) | Job history, owners, proxies, missing emails, AG jobs, MI limits | ~7 min |
| 5 | [Automate tasks in Azure](<05 - Automate tasks in Azure.md>) | Elastic jobs, Azure Automation runbooks, Logic Apps, which tool when | ~9 min |
| 6 | [Alerts, monitoring and troubleshooting automated tasks](<06 - Alerts, monitoring and troubleshooting automated tasks.md>) | Azure Monitor alerts, action groups, job history, Log Analytics | ~5 min |

## Words you will meet
| Word | Simple meaning |
|------|----------------|
| Infrastructure as Code (IaC) | Describing servers and databases in a code file, so you can build them again and again the same way |
| ARM template | A JSON file that describes Azure resources |
| Bicep | A simpler language for the same thing; it turns into ARM JSON when you deploy |
| SQL Server Agent | The built-in job scheduler of SQL Server |
| Operator | A person or email group that gets job emails |
| Database Mail | The SQL Server feature that sends email |
| Elastic job | A T-SQL job that runs on many Azure SQL databases at once |
| Runbook | A PowerShell or Python script run by Azure Automation |
| Logic App | A low-code workflow, drawn in a designer, that reacts to events |
| Action group | The list of "who to tell and what to do" when an Azure alert fires |

## Exam skills covered
- [ ] Manage schedules for regular maintenance jobs → 3
- [ ] Configure job alerts and notifications → 3, 4
- [ ] Troubleshoot SQL Server Agent jobs → 4
- [ ] Automate deployment by using ARM and Bicep templates → 1
- [ ] Automate deployment by using Azure PowerShell → 1
- [ ] Automate deployment by using Azure CLI → 1
- [ ] Monitor and troubleshoot deployments → 2
- [ ] Create and configure elastic jobs → 5
- [ ] Create and configure database tasks by using automation → 5
- [ ] Configure alerts and notifications on database tasks → 3, 6
- [ ] Troubleshoot automated database tasks → 6
