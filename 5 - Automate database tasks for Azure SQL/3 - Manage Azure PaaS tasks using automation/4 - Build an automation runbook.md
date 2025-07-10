## ✅ **Key Steps to Create an Automation Runbook**

---

### 1. **Create an Azure Automation Account**

An **Automation Account** is the starting point. It contains your:

- Runbooks
- Modules
- Credentials
- Schedules
- Hybrid workers (optional)

➡ **How to create it:**

- Go to Azure Portal → **Automation Accounts → + Create.**
- Fill in:

  - Name
  - Subscription
  - Resource Group
  - Region

---

### 2. **Import Required PowerShell Modules**

Runbooks use **PowerShell modules** to run Azure commands.

For Azure SQL automation, you need:

- `Az.Accounts` (for authentication)
- `Az.Sql` (for managing SQL databases)

➡ Steps:

- In your Automation Account → Go to **Shared Resources → Modules**
- Search the module from the **Gallery**
- Import the **Az.Accounts** module first
- Then import **Az.Sql**

---

### 3. **(Optional) Create a Credential**

A **credential** securely stores a username and password used by your runbook.

➡ Example Use:

- Storing SQL Database login (`SQLUser`) for running queries
- Go to **Shared Resources → Credentials → + Add a Credential**

---

### 4. **Create the Runbook**

➡ Steps:

- Go to your Automation Account → **Process Automation → Runbooks → + Create a runbook**
- Provide:

  - Name: Example, `QuerySQLDB`
  - Type: **PowerShell**
  - Runtime: PowerShell 7.2 recommended

- The **PowerShell editor** opens for writing your script.

---

### 5. **Write the Runbook Script**

The example script:

- Logs in to Azure using the **system-managed identity**
- Retrieves an Azure SQL Database name
- Connects to the database using stored credentials
- Runs a query on the database
- Outputs the results

```powershell
# Log in to Azure using system identity
try {
    Write-Output "Logging in to Azure..."
    Connect-AzAccount -Identity
} catch {
    Write-Error -Message $_.Exception
    throw $_.Exception
}

# Get SQL Database name
$dbname = (Get-AzSQLDatabase -ResourceGroupName 'SQLDB' -ServerName 'GSData' -DatabaseName 'GSData').DatabaseName

# Set Azure SQL Server FQDN
$AzureSQLServerName = $dbname + ".database.windows.net"

# Get SQL credentials from Automation
$Cred = Get-AutomationPSCredential -Name "SQLUser"

# Run SQL Query
$SQLOutput = $(Invoke-Sqlcmd -ServerInstance $AzureSQLServerName `
                             -Username $Cred.UserName `
                             -Password $Cred.GetNetworkCredential().Password `
                             -Database $dbname `
                             -Query "SELECT * FROM INFORMATION_SCHEMA.TABLES" `
                             -Verbose) 4>&1

Write-Output $SQLOutput
```

---

### 6. **Key Parts of the Script**

| Function                      | Purpose                                                 |
| ----------------------------- | ------------------------------------------------------- |
| `Connect-AzAccount -Identity` | Logs in using the Automation Account’s managed identity |
| `Get-AzSqlDatabase`           | Gets your Azure SQL Database details                    |
| `Get-AutomationPSCredential`  | Retrieves saved username/password                       |
| `Invoke-Sqlcmd`               | Executes a SQL query on your Azure SQL Database         |

---

### 7. **Test Your Runbook**

Click **Test pane** in the Azure Portal to:

- Run your script in a controlled environment
- See output and errors
- Confirm Azure Automation permissions are working

---

### 8. **⚠️ Important Points**

| Consideration        | Description                                                                                                                         |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Restart behavior** | If the runbook stops, it restarts from the beginning. Design your scripts to be restart-safe.                                       |
| **Permissions**      | Ensure the Automation Account's **system-managed identity** has appropriate **RBAC roles** on your SQL database and resource group. |
| **Testing**          | Test scripts both locally and in Azure Automation, as they might behave differently due to the execution environment.               |

---

### ✅ **Use Case Example**

Automate a daily query to check your SQL database structure:

- Run this runbook daily using a schedule
- Send the output to email (by adding email logic)
- Stop and start VMs before and after the job, if needed

---
