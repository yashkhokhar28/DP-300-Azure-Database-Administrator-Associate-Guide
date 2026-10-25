## ✔️ **1. Deploying from the Azure Portal (Web Interface)**

- **Go to Azure Portal > SQL Databases > Create**
- Fill in the following:

  - **Resource Group:** A logical group for your resources.
  - **Database Name:** Your database name.
  - **Server:** Create or select a logical SQL server that will host the database.
  - **Elastic Pool:** Optionally add the database to an elastic pool (a shared resource group for multiple databases).
  - **Compute + Storage:** Choose your performance level. For example:

    - **Serverless:** Auto-scales based on usage (you pay per second when it's running).
    - **Provisioned:** Fixed compute power billed hourly.

💡 **Example:**

- Gen5 server, 2 vCores, 32 GB storage.
- Choose **Serverless** to save costs if the database is idle sometimes.

---

## ✔️ **2. Deploying Using PowerShell**

You can automate the creation of:

- A resource group
- A logical SQL server
- A SQL Database
- A firewall rule to allow connections.

💻 **Sample Steps:**

- `New-AzResourceGroup`: Create a resource group.
- `New-AzSqlServer`: Create a logical SQL server.
- `New-AzSqlServerFirewallRule`: Allow access to the server from specified IP addresses.
- `New-AzSqlDatabase`: Create the database with a sample schema (`AdventureWorksLT` in this case).

---

## ✔️ **3. Deploying Using Azure CLI (Command Line for Bash/Linux/Windows)**

Similar steps as PowerShell but in Bash syntax:

- `az group create`: Create a resource group.
- `az sql server create`: Create a logical server.
- `az sql server firewall-rule create`: Allow IP access.
- `az sql db create`: Create the database.

💡 **Key point:** Uses environment variables like `$resourceGroupName`, `$servername`.

---

## ✔️ **4. Deploying Using ARM Templates (Infrastructure as Code)**

- ARM = **Azure Resource Manager** template (JSON file).
- You can automate deployment by specifying the configuration in a template.
- Microsoft provides pre-built templates on GitHub: [Azure Quickstart Templates](https://github.com/Azure/azure-quickstart-templates).

⚙️ **Example flow:**

- Run `New-AzResourceGroupDeployment`
- Pass in the template URL and parameters like admin username/password.

---

## ✅ **Summary of Deployment Methods**

| Deployment Method | Use Case                                                       |
| ----------------- | -------------------------------------------------------------- |
| Azure Portal      | Easy, manual setup                                             |
| PowerShell        | Scripting for Windows admins, automation                       |
| Azure CLI         | Scripting on Linux/Mac/CloudShell, automation                  |
| ARM Template      | Full automation for large deployments (Infrastructure as Code) |

---
