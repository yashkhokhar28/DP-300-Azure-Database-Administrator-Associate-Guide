## 🔐 **What is the Firewall in Azure SQL Database?**

A **firewall in Azure SQL Database** controls **who can connect to your database**. It works by **allowing or blocking access** based on **IP addresses** or **virtual network rules**, similar to how firewalls work in networks.

➡️ **All connection requests (whether from the internet or Azure services) pass through the firewall first.**

---

## 🔑 **Key Types of Firewall Rules**

| Firewall Type               | Scope                           | Configurable From                 | Usage Example                                              |
| --------------------------- | ------------------------------- | --------------------------------- | ---------------------------------------------------------- |
| **Server-level firewall**   | All databases on the SQL server | Portal, PowerShell, CLI, REST or T-SQL in `master` | Allow IP 192.168.1.1 to access all databases 🔄 |
| **Database-level firewall** | A specific database only        | T-SQL in that database            | Allow IP 192.168.1.1 to access just the `SalesDB` database |

---

## 🔍 **How Firewall Rules Work**

When a client tries to connect:

1. **Azure checks for a database-level firewall rule** for that IP and database.
2. If not found, it checks the **server-level firewall rules**.
3. If no matching rule exists, the connection is **blocked**.

If you're using **SQL Server Management Studio (SSMS)**, you'll be prompted to add your IP as a firewall rule if access is denied (Azure Data Studio was retired on 28 Feb 2026). 🔄

---

## ⚙️ **Server-level Firewall Rules**

- Apply to **all databases** on the server.
- Can be managed using:

  - Azure Portal
  - `sp_set_firewall_rule` in the **master** database.

✔️ Example: Allow your company's public IP (e.g., 52.120.14.22) to access all databases.

⚠️ Note:
There is an **“Allow Azure services to access this server”** option. If enabled, Azure services (like Azure App Service) can access your database but this counts as **one firewall rule**. It creates the `0.0.0.0` rule **AllowAllWindowsAzureIps**, which allows **all** Azure services, **including other customers' subscriptions**. Prefer VNet rules or private endpoints. 🔄

- Limits: up to **256** server-level and **256** database-level IP rules. These IP rules apply to **Azure SQL Database** only (Managed Instance uses NSGs). 🔄

---

## ⚙️ **Database-level Firewall Rules**

- Apply **only to one database**.
- Can be managed using:

  - `sp_set_database_firewall_rule` in the **target database**.

✔️ Example: Allow an external partner's IP to access just your reporting database.

---

## 🌐 **Virtual Network Endpoints**

Instead of allowing access from any public IP, you can:

- Allow access **only from an Azure Virtual Network (VNet)**.
- These are called **virtual network service endpoints**.

Limitations:

- Service endpoints apply to one **Azure region**.
- Your VNet must have outbound access to the public IP of the Azure SQL Database.

---

## 🔒 **Private Link (Private Endpoint)**

The most secure option:

- Creates a **private IP address** inside your Azure VNet.
- Traffic **never leaves Azure’s private network** (no public internet).
- Supports cross-region connections and ExpressRoute.

✔️ Recommended for high-security production databases.

- **Public network access = Disable**: only private endpoint connections are allowed, and firewall rules can't be added or edited (error 42101). 🔄
- **Network security perimeter** (preview for Azure SQL Database) groups SQL DB with other PaaS resources (Storage, Key Vault) and controls inbound and outbound traffic. It starts in **transition** mode (formerly learning mode), then you switch it to **Enforced**. 🔄
- **Connection policy** (server → Networking → Connectivity): **Redirect** (recommended; client connects directly to the node, needs outbound ports **11000–11999**), **Proxy** (all traffic through the gateway on **1433**), **Default** = Redirect inside Azure, Proxy from outside Azure. 🔄

---

## ✅ **Summary of Best Practices**

| Scenario                                     | Recommended Firewall Type                |
| -------------------------------------------- | ---------------------------------------- |
| Allow entire company to access all databases | Server-level firewall                    |
| Allow an external team to access 1 database  | Database-level firewall                  |
| Secure connection from an Azure VNet         | Virtual Network Endpoint or Private Link |
| Require the highest level of security        | Private Link                             |

---
