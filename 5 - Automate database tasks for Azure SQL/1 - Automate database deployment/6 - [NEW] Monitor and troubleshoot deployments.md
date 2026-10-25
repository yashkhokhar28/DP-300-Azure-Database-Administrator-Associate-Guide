### ✅ **Monitor and Troubleshoot ARM / Bicep Deployments (Azure SQL)**

---

## 🧭 **Three Kinds of Deployment Errors**

| Error type | When it happens | Where you see it |
| --- | --- | --- |
| **Validation (syntax) errors** | Before deployment — bad JSON/Bicep syntax | VS Code (Bicep / ARM Tools extension), `bicep build` |
| **Preflight validation errors** | Command runs, but **nothing is deployed** (e.g., bad parameter value, SKU not available) | **Activity log** only — **not** in deployment history |
| **Deployment errors** | Passed validation, **failed while deploying** a resource | **Deployment history** + **Activity log** |

> 💡 Bicep with syntax errors doesn't compile to JSON → it doesn't even reach the Activity log.

---

## 🧪 **Test Before You Deploy: Validate & What-If**

```powershell
# Preflight validation (no changes)
Test-AzResourceGroupDeployment -ResourceGroupName rg-sql -TemplateFile main.bicep

# What-if: preview changes (Create / Delete / Modify / NoChange / Ignore / NoEffect / Deploy)
New-AzResourceGroupDeployment -ResourceGroupName rg-sql -TemplateFile main.bicep -WhatIf

# Preview + prompt to continue
New-AzResourceGroupDeployment -ResourceGroupName rg-sql -TemplateFile main.bicep -Confirm
```

```bash
az deployment group validate --resource-group rg-sql --template-file main.bicep
az deployment group what-if  --resource-group rg-sql --template-file main.bicep
az deployment group create   --resource-group rg-sql --template-file main.bicep --confirm-with-what-if   # or -c
```

- **What-if** = predicts changes, makes **no changes**; same permissions as a real deployment.
- **Delete** change type only appears in **complete mode**.
- Known "noise": properties using **`reference()`** always show as changing; defaults may show as deleted.
- Result formats: **FullResourcePayloads** (default) vs **ResourceIdOnly**.

---

## 📜 **Deployment History**

- Every deployment to a scope (RG / subscription / MG / tenant) is recorded with a **correlation ID** (give this to Microsoft Support).
- **Portal:** Resource group → **Settings > Deployments** → select deployment → **Error details** / **Operation details**.

```powershell
# List / get a deployment and its correlation ID
Get-AzResourceGroupDeployment -ResourceGroupName rg-sql
(Get-AzResourceGroupDeployment -ResourceGroupName rg-sql -DeploymentName sqlDeploy01).CorrelationId

# Per-resource operations -> find the failed one and its message
Get-AzResourceGroupDeploymentOperation -ResourceGroupName rg-sql -DeploymentName sqlDeploy01 |
    Where-Object { $_.ProvisioningState -eq "Failed" } | Select-Object -ExpandProperty StatusMessage
```

```bash
az deployment group list --resource-group rg-sql -o table
az deployment group show --resource-group rg-sql --name sqlDeploy01 --query properties.correlationId
az deployment operation group list --resource-group rg-sql --name sqlDeploy01 \
  --query "[?properties.provisioningState=='Failed'].properties.statusMessage.error"
```

| Scope | PowerShell | CLI |
| --- | --- | --- |
| Resource group | `Get-AzResourceGroupDeployment(Operation)` | `az deployment group …` / `az deployment operation group list` |
| Subscription | `Get-AzDeployment` / `Get-AzSubscriptionDeployment` | `az deployment sub …` |
| Management group | `Get-AzManagementGroupDeployment` | `az deployment mg …` |
| Tenant | `Get-AzTenantDeployment` (not shown in portal) | `az deployment tenant …` |

### ⚠️ **The 800-deployment limit**

- Each scope keeps **max 800 deployments** in history.
- **Automatic deletion**: when history passes **700**, ARM deletes the **oldest** until it's down to **600** (what-if and validate also trigger this).
- If already at 800 → next deployment fails with **`DeploymentQuotaExceeded`**; auto-cleanup starts, retry shortly or delete old entries (`Remove-AzResourceGroupDeployment` / `az deployment group delete`).
- Auto-delete **won't work** if a **CanNotDelete lock** exists on the RG/subscription, or the deployer lacks `Microsoft.Resources/deployments/delete`.
- Deleting history **doesn't delete resources**.

---

## 📋 **Activity Log**

- Subscription-level log of **control-plane (ARM) operations** — who created/updated/deleted what, and failures.
- Shows **preflight failures** that never appear in deployment history.
- Portal: Resource group → **Activity log** → filter by *Status = Failed* / operation. Can be routed to Log Analytics (`AzureActivity` table) for KQL and **activity log alerts**.

---

## 🐞 **Debug Logging**

```powershell
New-AzResourceGroupDeployment -Name sqlDebug01 -ResourceGroupName rg-sql `
  -TemplateFile main.bicep -DeploymentDebugLogLevel All   # All | RequestContent | ResponseContent
```

```bash
# Read the captured request/response (PowerShell cmdlets don't output these properties)
az deployment operation group list --resource-group rg-sql --name sqlDebug01 --query "[].properties.request"
```

- **Only PowerShell can enable it** — Azure CLI **can't** enable debug logging (but can read `request`/`response`).
- Applies only to the **main template**; nested templates need `debugSetting` in `Microsoft.Resources/deployments`.
- ⚠️ Can expose **secrets** (passwords, `listKeys`) → **delete the debug deployment history** afterwards.

---

## ❌ **Common Errors (Azure SQL flavoured)**

| Error code | Cause | Fix |
| --- | --- | --- |
| **InvalidTemplate / InvalidTemplateDeployment** | Syntax error, bad expression, or preflight failure wrapped | Fix template; run `validate` / `bicep build` |
| **DeploymentFailed** | Generic wrapper | Drill into **inner error code** in operation details |
| **DeploymentQuotaExceeded** | 800 deployments in history | Delete old deployments / remove lock |
| **ResourceNotFound / NotFound / ParentResourceNotFound** | Database deployed before its server (missing dependency) or wrong `reference()` | Add **`dependsOn`** (ARM) / use **`parent:`** or symbolic reference (Bicep) |
| **InvalidTemplateCircularDependency** | Resources depend on each other | Remove unnecessary `dependsOn` |
| **Conflict** / **AnotherOperationInProgress** | Resource busy (e.g., scaling DB while another op runs) | Wait and retry |
| **SkuNotAvailable** / **RegionDoesNotAllowProvisioning** | Service tier / region not offered to subscription | Pick another SKU/region; check `az sql db list-editions -l <region> --available`; request region access |
| **Server name taken** (e.g., `NameAlreadyExists`) | Logical server names are **globally unique** (`<name>.database.windows.net`): **1–63 chars, lowercase letters, numbers, hyphens, no leading/trailing hyphen** | Use `uniqueString(resourceGroup().id)` in the name |
| **ResourceQuotaExceeded / OperationNotAllowed** | Subscription/regional quota (e.g., vCores, servers per region) | Request quota increase or resize |
| **AuthorizationFailed / RequestDisallowedByPolicy** | Missing RBAC role / Azure Policy blocks (e.g., required tags, allowed SKUs) | Grant role / comply with policy |
| **MissingSubscriptionRegistration** | `Microsoft.Sql` provider not registered | `Register-AzResourceProvider -ProviderNamespace Microsoft.Sql` |
| **InvalidParameter** | e.g., admin password not complex enough | Fix parameter value |

---

## 🔁 **Deployment Modes & Rollback on Error**

| Mode | Behavior |
| --- | --- |
| **Incremental** (default, recommended) | Leaves resources not in template untouched — **but properties of resources in the template are fully re-applied** (omitted properties reset to defaults!) |
| **Complete** | **Deletes** resources in the RG not in the template. **Being gradually deprecated** → Microsoft recommends **deployment stacks** for deletes. Not supported at subscription scope, nested templates, or in the portal |

```powershell
# Roll back to last successful deployment if this one fails
New-AzResourceGroupDeployment -Name sqlDeploy02 -ResourceGroupName rg-sql -TemplateFile main.json -RollbackToLastDeployment
# ...or to a named successful deployment
New-AzResourceGroupDeployment -Name sqlDeploy02 -ResourceGroupName rg-sql -TemplateFile main.json -RollBackDeploymentName sqlDeploy01
```

```bash
az deployment group create -g rg-sql -n sqlDeploy02 --template-file main.json --rollback-on-error            # last successful
az deployment group create -g rg-sql -n sqlDeploy02 --template-file main.json --rollback-on-error sqlDeploy01
```

⚠️ Rollback **redeploys the earlier deployment in _complete_ mode** with its original parameters → can **delete** resources added since. **Resource group scope only**, root-level only, needs **unique deployment names**, and **doesn't roll back data**.

---

## ✅ **Summary**

| Need | Use |
| --- | --- |
| Catch errors before deploying | `Test-AzResourceGroupDeployment` / `az deployment group validate` |
| Preview changes | `-WhatIf` / `-Confirm` / `az deployment group what-if` / `-c` |
| See what failed | Deployment history → **operations** → status message |
| See preflight/control-plane failures | **Activity log** |
| See raw request/response | `-DeploymentDebugLogLevel All` (PowerShell only) |
| History full | `DeploymentQuotaExceeded` (800 limit) |
| Auto-recover | `-RollbackToLastDeployment` / `--rollback-on-error` |

---

## 🎯 **Exam Tips / Traps**

- **Preflight errors are NOT in deployment history** → check the **Activity log**.
- **Database created before server** → `ResourceNotFound`/`ParentResourceNotFound` → add **`dependsOn`** / Bicep **`parent`**.
- **"Server name already exists"** → server names are **globally unique DNS names** → `uniqueString()`.
- **"Show what will change without deploying"** → **what-if** (not `validate`, which only checks validity).
- **Debug logging can't be enabled from Azure CLI** — only PowerShell `-DeploymentDebugLogLevel`.
- **800 deployments per scope**; auto-cleanup at **700 → 600**; blocked by **CanNotDelete locks**.
- **Incremental** ≠ partial update: unspecified properties revert to defaults.
- **Complete mode** deletes extra resources; **rollback on error uses complete mode**; Microsoft now steers deletes to **deployment stacks**.
- Give Support the **correlation ID**.

---

> Source: https://learn.microsoft.com/en-us/azure/azure-resource-manager/troubleshooting/find-error-code, https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-history, https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-history-deletions, https://learn.microsoft.com/en-us/azure/azure-resource-manager/troubleshooting/common-deployment-errors, https://learn.microsoft.com/en-us/azure/azure-resource-manager/troubleshooting/enable-debug-logging, https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-what-if, https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-modes, https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/rollback-on-error, https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/resource-name-rules, https://learn.microsoft.com/en-us/cli/azure/sql/db
