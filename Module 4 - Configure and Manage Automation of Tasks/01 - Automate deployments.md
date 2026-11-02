# Automate deployments

> **Exam skills:** Automate deployment by using ARM and Bicep templates, Automate deployment by using Azure PowerShell, Automate deployment by using Azure CLI
> **Reading time:** ~8 min

## The big idea
In a data centre, a new server means racks and cables. In Azure, everything is created through an API (a doorway programs use to talk to Azure). So you can write your servers and databases down as code. This is **Infrastructure as Code (IaC)**. It is like a recipe card: anyone who follows it gets the same cake every time.

Why IaC helps:
- The same file builds **Dev, Test and Prod** the same way.
- Fewer human mistakes than clicking in the portal.
- You keep the files in **source control** (GitHub, Azure Repos). You can see every change and go back to an older version.
- A **CI/CD pipeline** (Azure Pipelines or GitHub Actions) can deploy the file automatically on every change.

**Example:** A school app needs the same database in 3 environments. One Bicep file plus 3 small parameter files builds all three.

## Two deployment models

| | Declarative | Imperative |
|---|---|---|
| You write | The **end result** you want | **Each step**, in order |
| Who decides the order | Azure (it reads the dependencies) | You |
| Tools | **ARM templates**, **Bicep** | **Azure PowerShell**, **Azure CLI** |
| Best for | Big, repeatable deployments | Small jobs, quick changes, checks |

**Example:** Declarative is telling a taxi "take me to the airport". Imperative is giving every turn: "left, then right, then straight".

## ARM templates
- An **ARM template** (Azure Resource Manager template) is a **JSON** file that lists resources and their settings.
- It supports **parameters**, so one file works with different values (region, size, names).
- ARM works out the **order**. For example, it creates the SQL server before the database. In JSON you state this order with **`dependsOn`**.
- You can create a resource in the portal and **export** it as a template.
- You deploy templates with the portal, PowerShell, Azure CLI or Azure Pipelines.

## Bicep
- **Bicep** is Microsoft's **recommended** language for templates.
- It is much shorter and easier to read than JSON. A storage account is about 6 lines of Bicep but about 20 lines of JSON.
- A `.bicep` file is **turned into ARM JSON** when you deploy. Anything ARM can do, Bicep can do.
- To turn an exported JSON template into Bicep, use **`bicep decompile`** (or `az bicep decompile`).
- In Bicep, a child resource uses **`parent:`**. Bicep then works out the dependency for you.

**Example (Bicep): a SQL server and one database**
```bicep
@secure()
param adminPassword string
resource srv 'Microsoft.Sql/servers@2021-11-01' = {
  name: 'shop-sql-${uniqueString(resourceGroup().id)}'
  location: resourceGroup().location
  properties: {
    administratorLogin: 'sqladmin'
    administratorLoginPassword: adminPassword
  }
}
resource db 'Microsoft.Sql/servers/databases@2021-11-01' = {
  parent: srv
  name: 'shopdb'
  location: resourceGroup().location
  sku: { name: 'S0' }
}
```
`@secure()` hides the password from logs. `uniqueString()` makes a server name that nobody else has used.

**The same database in ARM JSON (only the part that differs):**
```json
{
  "type": "Microsoft.Sql/servers/databases",
  "apiVersion": "2021-11-01",
  "name": "[concat(parameters('serverName'), '/shopdb')]",
  "location": "[resourceGroup().location]",
  "sku": { "name": "S0" },
  "dependsOn": [ "[resourceId('Microsoft.Sql/servers', parameters('serverName'))]" ]
}
```
In JSON you must write `dependsOn` yourself. In Bicep, `parent: srv` does it.

## Azure PowerShell
- PowerShell is a shell and scripting language. It runs on **Windows, Linux and macOS**.
- The **Az module** manages Azure. **Az.Sql** is the part for Azure SQL Database and Managed Instance.
- Commands use **Verb-Noun** names: `New-AzSqlServer`, `Get-AzSqlDatabase`.
- Run it on your PC, in **Azure Cloud Shell** (in the browser, nothing to install), or in Docker.

**Example (PowerShell): create a server and a database**
```powershell
Connect-AzAccount
New-AzResourceGroup -Name shopRG -Location eastus
New-AzSqlServer -ResourceGroupName shopRG -ServerName shop-sql-01 `
  -Location eastus -SqlAdministratorCredentials (Get-Credential)
New-AzSqlDatabase -ResourceGroupName shopRG -ServerName shop-sql-01 `
  -DatabaseName shopdb -RequestedServiceObjectiveName S0
```
- `New-AzSqlInstance` creates a Managed Instance. Note: **Gen4 hardware is retired**; use Gen5 or newer.

**Deploy a template with PowerShell:**
```powershell
New-AzResourceGroupDeployment -ResourceGroupName shopRG `
  -TemplateFile ./main.bicep -TemplateParameterFile ./prod.parameters.json
```
- Azure PowerShell needs the **Bicep CLI installed by hand**.

## Azure CLI
- The **Azure CLI** (command-line interface) also runs everywhere, including Cloud Shell.
- It works in **Cmd, Bash and PowerShell**. Azure PowerShell works only in PowerShell.
- Command shape: `az <group> <command> --parameter value`.
- Some commands for Azure Database for PostgreSQL and MySQL exist **only in the CLI**.

**Example (Azure CLI): the same server and database**
```bash
az login
az group create --name shopRG --location eastus
az sql server create --name shop-sql-01 --resource-group shopRG \
  --location eastus --admin-user sqladmin --admin-password "$PASS"
az sql server firewall-rule create --resource-group shopRG --server shop-sql-01 \
  -n AllowAzure --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
az sql db create --resource-group shopRG --server shop-sql-01 --name shopdb --service-objective S0
```
The firewall rule `0.0.0.0` to `0.0.0.0` means "allow Azure services".

**Deploy a template with the CLI:**
```bash
az deployment group create --resource-group shopRG --template-file main.bicep
az deployment group create --resource-group shopRG --template-uri "https://<storage>/main.json" --query-string $sasToken
```
- The CLI **installs Bicep automatically**.
- A **local** `.bicep` file deploys directly. You can also pass `--parameters main.bicepparam`.
- **Neither tool can deploy a remote `.bicep` file.** For a file in storage or on GitHub, build it to JSON first, then deploy the JSON. Remote JSON templates work, with a **SAS token** (a time-limited access key) for private storage.
- CLI scopes: `az deployment group` (resource group), `sub` (subscription), `mg` (management group), `tenant`.

## Compare at a glance
| Task | Azure CLI | PowerShell |
|---|---|---|
| Sign in | `az login` | `Connect-AzAccount` |
| Switch subscription | `az account set --subscription` | `Set-AzContext -Subscription` |
| Create SQL server | `az sql server create` | `New-AzSqlServer` |
| Create database | `az sql db create` | `New-AzSqlDatabase` |
| Deploy template | `az deployment group create` | `New-AzResourceGroupDeployment` |
| Installs Bicep for you | Yes | No |

## IaaS vs PaaS deployments
On an **Azure VM** you must also script the SQL Server install and settings, and patch it. On **SQL Database / Managed Instance**, Azure installs and patches SQL; you only choose database settings.

## Remember for the exam
- "Describe the end state, repeatable, in source control" → **ARM or Bicep** (declarative).
- "Quick change or check a setting" → **PowerShell or CLI** (imperative).
- **Bicep turns into ARM JSON** at deploy time. Bicep is the recommended language.
- JSON uses **`dependsOn`**. Bicep uses **`parent`** or a symbolic reference.
- **Remote `.bicep` files cannot be deployed** by CLI or PowerShell. Build to JSON first.
- PowerShell = `New-AzResourceGroupDeployment`. CLI = `az deployment group create`.

## Quick check
1. You must deploy a Bicep file stored in a storage account using the Azure CLI. What do you do?
   <details><summary>Answer</summary>Build it to ARM JSON first, then deploy the JSON with --template-uri. The CLI cannot deploy remote .bicep files.</details>
2. Your team wants the same SQL setup in Dev, Test and Prod, kept in Git. Declarative or imperative?
   <details><summary>Answer</summary>Declarative (Bicep or ARM). You describe the end state once and reuse it.</details>
3. In an ARM JSON template, the database is created before its server and fails. What is missing?
   <details><summary>Answer</summary>A dependsOn on the server. In Bicep you would use parent.</details>
