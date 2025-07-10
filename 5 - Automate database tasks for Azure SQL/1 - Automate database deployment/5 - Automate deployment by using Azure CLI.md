## ✅ **Why Automate with Azure CLI?**

Automation:

- Saves time by avoiding manual configuration.
- Allows you to **scale deployments** easily across environments.
- Lets you **re-use scripts** for repeated tasks.
- Makes troubleshooting easier in environments where **GUI tools aren’t available**.

Automation is essential for **businesses of all sizes**, not just large enterprises.

---

## 🔧 **What is Azure CLI?**

Azure CLI (**Command-Line Interface**) is a **cross-platform** tool to create and manage Azure resources from:

- **Linux**, **macOS**, or **Windows**
- Azure **Cloud Shell** (browser-based terminal)
- **Docker containers**

You can run Azure CLI commands **interactively** in a terminal or automate them inside a script.

---

## 🛠️ **Azure CLI Command Syntax**

Basic command structure:

```
az <resource> <command> --parameter-name parameter-value
```

Example to change Azure subscription:

```bash
az account set --subscription "my subscription name"
```

---

## ⚙️ **Azure CLI vs. PowerShell**

| Shell Environment     | Azure CLI | Azure PowerShell |
| --------------------- | --------- | ---------------- |
| Cmd (Windows Command) | ✅        |                  |
| Bash (Linux/Mac)      | ✅        |                  |
| Windows PowerShell    | ✅        | ✅               |
| PowerShell Core       | ✅        | ✅               |

### ✔ Use **Azure CLI** if:

- You prefer **bash-style scripting**.
- You work mostly on **Linux/macOS**.
- You want faster syntax for quick actions.

✔ Use **PowerShell** if:

- You are on **Windows** and already use PowerShell.
- You prefer **verb-noun syntax** (e.g., `New-AzSqlServer`).
- You want advanced scripting support in the PowerShell ecosystem.

---

## 🔑 **Common Azure CLI vs. PowerShell Commands**

| Task                | Azure CLI                       | PowerShell                    |
| ------------------- | ------------------------------- | ----------------------------- |
| Sign in             | `az login`                      | `Connect-AzAccount`           |
| List subscriptions  | `az account list`               | `Get-AzSubscription`          |
| Switch subscription | `az account set --subscription` | `Set-AzContext -Subscription` |
| List VMs            | `az vm list`                    | `Get-AzVM`                    |
| Create SQL server   | `az sql server create`          | `New-AzSqlServer`             |

---

## ▶️ **Example: Deploy SQL Database Using Azure CLI**

```bash
let "randomIdentifier=$RANDOM*$RANDOM"

resourceGroup="<your resource group>"
location="<your location preference>"
server="dp300-sql-server-$randomIdentifier"
login="sqladmin"
password="Pa$$w0rD-$randomIdentifier"

# Create SQL Server
az sql server create \
    --name $server \
    --resource-group $resourceGroup \
    --location "$location" \
    --admin-user $login \
    --admin-password $password

# Create firewall rule to allow Azure access
az sql server firewall-rule create \
    --resource-group $resourceGroup \
    --server $server \
    -n AllowYourIp \
    --start-ip-address 0.0.0.0 \
    --end-ip-address 0.0.0.0
```

---

## 📂 **Deploy ARM Templates Using Azure CLI and PowerShell**

### With **PowerShell**:

```powershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
 -TemplateFile c:\MyTemplates\azuredeploy.json `
 -TemplateParameterFile c:\MyTemplates\storage.parameters.json
```

### With **Azure CLI**:

```bash
az deployment group create --resource-group ExampleResourceGroup --template-file '/path/template.json'
```

You can also deploy templates stored remotely:

```bash
az deployment group create \
  --name linkedTemplateWithRelativePath \
  --resource-group myResourceGroup \
  --template-uri "https://<storage-account>.blob.core.windows.net/template-staging/mainTemplate.json" \
  --query-string $sasToken
```

> ⚠️ **Note:** Azure CLI **cannot directly deploy remote Bicep files**.
> You must first convert Bicep to JSON using the **Bicep CLI**, then deploy the JSON file.

---

## 🔑 **Key Takeaways**

| Feature                    | Azure CLI                                              |
| -------------------------- | ------------------------------------------------------ |
| Deployment Scope           | Resource group, subscription, management group, tenant |
| Supports ARM Template      | ✅                                                     |
| Remote Template Deployment | ✅ with SAS token for secure storage access            |
| Remote Bicep Deployment    | ❌ Convert Bicep → JSON first                          |
| Platform                   | Cross-platform (Windows, Linux, Mac, Cloud Shell)      |

---
