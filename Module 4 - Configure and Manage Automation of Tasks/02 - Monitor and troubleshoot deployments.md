# Monitor and troubleshoot deployments

> **Exam skills:** Monitor and troubleshoot deployments
> **Reading time:** ~8 min

## The big idea
A template can fail before it starts, or halfway through. You need to know **where to look** for each kind of failure. Think of posting a parcel. The post office can refuse it at the counter (bad address). Or it can get lost on the way. You check different places for each problem.

## Three kinds of deployment errors
| Error type | When it happens | Where you see it |
|---|---|---|
| **Validation (syntax) error** | Before anything is sent. Bad JSON or Bicep. | VS Code (Bicep / ARM Tools extension), `bicep build` |
| **Preflight error** | The command runs, but **nothing is deployed**. Example: a bad parameter, or the SKU (size) is not offered in that region. | **Activity log only**. Not in deployment history. |
| **Deployment error** | Passed checks, but **failed while creating** a resource. | **Deployment history** and the activity log |

A Bicep file with syntax errors never becomes JSON. So it never reaches the activity log.

**Example:** A bank's script asks for SKU "S9" in a region that does not sell it. The command returns an error but no deployment appears under the resource group's Deployments. The DBA finds the failure in the **activity log**.

## Test before you deploy: validate and what-if
- **Validate** checks the template is valid. It does not say what will change.
- **What-if** shows what **would** change. It makes **no changes**. Change types: Create, Delete, Modify, NoChange, Ignore, NoEffect, Deploy.
- **Delete** only shows in **complete mode** (see below).

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName rg-sql -TemplateFile main.bicep   # validate
New-AzResourceGroupDeployment  -ResourceGroupName rg-sql -TemplateFile main.bicep -WhatIf
New-AzResourceGroupDeployment  -ResourceGroupName rg-sql -TemplateFile main.bicep -Confirm  # preview, then ask
```
```bash
az deployment group validate --resource-group rg-sql --template-file main.bicep
az deployment group what-if  --resource-group rg-sql --template-file main.bicep
az deployment group create   --resource-group rg-sql --template-file main.bicep --confirm-with-what-if  # or -c
```

## Deployment history
- Every deployment to a scope (resource group, subscription, management group, tenant) is saved with a **correlation ID**. Give this ID to Microsoft Support.
- **Portal:** resource group → **Settings > Deployments** → pick a deployment → **Error details** / **Operation details**.
- Each deployment has **operations** (one per resource). Find the failed one and read its status message.

```powershell
Get-AzResourceGroupDeployment -ResourceGroupName rg-sql
Get-AzResourceGroupDeploymentOperation -ResourceGroupName rg-sql -DeploymentName sqlDeploy01 |
  Where-Object { $_.ProvisioningState -eq "Failed" } | Select-Object -ExpandProperty StatusMessage
```
```bash
az deployment group list --resource-group rg-sql -o table
az deployment operation group list --resource-group rg-sql --name sqlDeploy01
```

### The 800 limit
- Each scope keeps at most **800 deployments** in history.
- When history goes above **700**, Azure deletes the oldest until it is down to **600**. What-if and validate also start this clean-up.
- If history is already at 800, the next deployment fails with **`DeploymentQuotaExceeded`**. Wait and retry, or delete old entries (`Remove-AzResourceGroupDeployment` / `az deployment group delete`).
- Auto clean-up **does not work** if there is a **CanNotDelete lock** on the resource group or subscription. It also fails if the user lacks the `Microsoft.Resources/deployments/delete` permission.

## Activity log
- The activity log records **control-plane** actions (create, update, delete of resources) in the subscription: who did what, and what failed.
- It shows **preflight failures** that never reach deployment history.
- Portal: resource group → **Activity log** → filter Status = Failed.
- You can send it to Log Analytics (table `AzureActivity`) and create **activity log alerts**.

## Debug logging
```powershell
New-AzResourceGroupDeployment -Name sqlDebug01 -ResourceGroupName rg-sql `
  -TemplateFile main.bicep -DeploymentDebugLogLevel All   # All | RequestContent | ResponseContent
```
- **Only PowerShell can turn it on.** The Azure CLI **cannot** turn it on, but it **can read** the saved request and response.
- It can save **secrets** such as passwords. **Delete that deployment from history** afterwards.

## Common errors for Azure SQL
| Error | Cause | Fix |
|---|---|---|
| **InvalidTemplate** | Syntax error or bad expression | Fix the file, run validate or `bicep build` |
| **DeploymentFailed** | A general wrapper | Read the **inner error** in the operation details |
| **ResourceNotFound / ParentResourceNotFound** | Database created before its server | Add **`dependsOn`** (JSON) or **`parent`** (Bicep) |
| **SkuNotAvailable / RegionDoesNotAllowProvisioning** | Tier or region not offered | Choose another SKU or region. Check with `az sql db list-editions -l <region> --available` |
| **NameAlreadyExists** | Server names are **global DNS names** (`<name>.database.windows.net`): 1–63 chars, lowercase letters, numbers, hyphens, no hyphen at start or end | Use `uniqueString(resourceGroup().id)` |
| **AuthorizationFailed / RequestDisallowedByPolicy** | Missing RBAC role (role-based access) or an Azure Policy blocks it (e.g. a required tag) | Grant the role or follow the policy |
| **MissingSubscriptionRegistration** | `Microsoft.Sql` provider not registered | `Register-AzResourceProvider -ProviderNamespace Microsoft.Sql` |

## Deployment modes and rollback
| Mode | What it does |
|---|---|
| **Incremental** (default) | Leaves resources that are **not** in the template alone. But resources that **are** in the template get **all** their properties re-applied. A property you left out goes back to its default. |
| **Complete** | **Deletes** resources in the resource group that are not in the template. Microsoft is slowly retiring it and recommends **deployment stacks** for deletes. Not supported at subscription scope, in nested templates, or in the portal. |

**Rollback on error** redeploys the last good deployment if this one fails:
```powershell
New-AzResourceGroupDeployment -Name sqlDeploy02 -ResourceGroupName rg-sql -TemplateFile main.json -RollbackToLastDeployment
```
```bash
az deployment group create -g rg-sql -n sqlDeploy02 --template-file main.json --rollback-on-error
```
- Add a deployment name (`-RollBackDeploymentName sqlDeploy01`, or a name after `--rollback-on-error`) to go back to a specific one.
- Rollback runs the old deployment in **complete mode**. It can **delete** resources added since then.
- Resource group scope only. Deployment names must be unique. It **does not roll back data**.

## Remember for the exam
- **Preflight errors are not in deployment history.** Look in the **activity log**.
- "Show what will change without deploying" → **what-if**, not validate.
- **Debug logging: PowerShell only** (`-DeploymentDebugLogLevel`). Delete the history afterwards.
- **800** per scope. Auto clean-up at **700 → 600**. Blocked by **CanNotDelete locks**.
- `DeploymentQuotaExceeded` = history full.
- Database before server → **`dependsOn` / `parent`**.
- "Server name already exists" → names are global → `uniqueString()`.
- **Incremental** resets omitted properties. **Complete** deletes extras. Rollback uses complete mode.
- Give Support the **correlation ID**.

## Quick check
1. A deployment command fails, but nothing shows under the resource group's Deployments. Where do you look?
   <details><summary>Answer</summary>The activity log. Preflight failures never reach deployment history.</details>
2. You want raw request and response details for a failed SQL deployment. Which tool turns this on?
   <details><summary>Answer</summary>Azure PowerShell with -DeploymentDebugLogLevel. The CLI cannot turn it on.</details>
3. Deployments now fail with DeploymentQuotaExceeded even though auto clean-up should run. What is a likely cause?
   <details><summary>Answer</summary>A CanNotDelete lock on the resource group or subscription stops automatic deletion of old history.</details>
