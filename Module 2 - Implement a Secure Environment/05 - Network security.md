# Network security

> **Exam skills:** Configure server- and database-level firewall rules, Configure private links and service endpoints
> **Reading time:** ~8 min

## The big idea
Before anyone can log in, their network traffic must reach the database. Network security decides **which networks and IP addresses** can even knock on the door. Think of a gated housing estate. The firewall is the gate guard with a list of allowed cars. A private link is a private tunnel from your garage straight into the estate.

## Firewall rules in Azure SQL Database
The Azure SQL Database firewall allows or blocks connections by **IP address** or by **virtual network rule**. Every connection goes through it first. By default, **everything is blocked**.

### Two levels

| Level | Scope | How to set it |
|-------|-------|---------------|
| **Server-level** | All databases on the logical server | Portal, PowerShell, CLI, REST, or T-SQL `sp_set_firewall_rule` in **master** |
| **Database-level** | One database only | **T-SQL only**: `sp_set_database_firewall_rule` in **that database** |

```sql
-- In master: allow the office IP to reach every database
EXEC sp_set_firewall_rule N'Office', '52.120.14.22', '52.120.14.22';
-- In SalesDB: allow a partner to reach only this database
EXEC sp_set_database_firewall_rule N'Partner', '203.0.113.5', '203.0.113.5';
```

### Order of checks
1. Azure checks the **database-level** rules for that database.
2. If none match, it checks the **server-level** rules.
3. If still none match, the connection is **blocked**.

**Example:** A school has 10 databases. All staff use the school office IP, so the DBA adds one **server-level** rule. An outside exam board needs only the `ResultsDB`. The DBA adds a **database-level** rule for its IP in `ResultsDB` only.

### Limits and tips
- Up to **256** server-level and **256** database-level IP rules.
- These IP firewall rules are for **Azure SQL Database**. **Managed Instance** uses **NSGs** (network security groups: allow/deny rules on a subnet) instead.
- In **SSMS**, if your IP is blocked, it offers to add a firewall rule for you. (Azure Data Studio was retired on 28 Feb 2026.)

### "Allow Azure services and resources to access this server"
- This creates a special rule `0.0.0.0` named **AllowAllWindowsAzureIps**.
- It lets **all** Azure services in, **including other customers' subscriptions**. It is not just your Azure resources.
- Prefer **VNet rules** or **private endpoints** instead.

## Virtual network service endpoints
A **service endpoint** lets traffic from a chosen **subnet** reach Azure SQL over the Microsoft backbone network. You then add a **virtual network rule** to the SQL firewall to allow that subnet.

- Access is controlled **by subnet**, not by IP.
- The database **still uses its public endpoint** (public address). Traffic is just routed better and tagged with the subnet.
- Applies within one **Azure region**.
- Easy to set up.

**Example:** A web app runs on VMs in subnet 10.0.0.0/24. The DBA turns on the SQL service endpoint for that subnet and adds a VNet rule. Only that subnet can connect; other Azure traffic is blocked.

## Private Link (private endpoint)
**Private Link** gives your database a **private IP address inside your VNet**. This is called a **private endpoint**.
- Traffic stays on Microsoft's private network. It never goes over the public internet.
- Works **across regions**, and from on-premises over **VPN** or **ExpressRoute**.
- It is the **most secure** choice, and best against **data exfiltration** (data being copied out to places it should not go).
- Set **Public network access = Disable** so only private endpoint connections are allowed. After that, you cannot add or edit firewall rules (error **42101**).

**Example:** A healthcare app must have no public exposure at all. The DBA creates a private endpoint so the database gets 10.1.2.5 inside the hospital VNet, then disables public network access.

## Other network options
- **VPN Gateway:** an encrypted tunnel over the internet from on-premises to Azure. Cheaper. Good for dev/test or smaller workloads.
- **ExpressRoute:** a **dedicated private line** from your datacentre to Azure. It does not use the internet. Fast, reliable, more expensive. Used for strict compliance.
- **Network security perimeter** (preview for Azure SQL Database): groups SQL Database with other PaaS resources (Storage, Key Vault) and controls traffic in and out. It starts in **transition** mode (formerly "learning" mode), then you switch to **Enforced**.

### Connection policy
Set on the server: **Networking → Connectivity**.
- **Redirect** (recommended): the client connects straight to the database node. Needs outbound ports **11000–11999**.
- **Proxy:** all traffic goes through the gateway on port **1433**.
- **Default:** Redirect for clients inside Azure, Proxy for clients outside Azure.

### SQL Server on Azure VMs
A VM is protected like any server: **NSGs** on the subnet, the **Windows Firewall** inside the VM, and VPN or ExpressRoute for hybrid links.

## SQL injection (basics)
**SQL injection** is an attack where a user types SQL code into an input box, and the app runs it.

**Example:** An app builds `SELECT * FROM Orders WHERE OrderId = ` + user input. An attacker types `25; DELETE FROM Orders;`. The database runs both statements and deletes all orders.

How to prevent it:
- **Parameterized queries** (best): `WHERE OrderId = @OrderId`. The input is treated as data, never as code.
- **Stored procedures** and **ORM** libraries (e.g. Entity Framework).
- **Validate input** (is it really a number?).
- **Least privilege** for the app's account, so a bad query can do little harm.
- **Web Application Firewall (WAF)** and **Microsoft Defender for SQL**, which alerts on SQL injection attempts (lesson 6).

Blocking words like "DELETE" is **not enough**. Attackers can hide commands in encoded or binary data.

## Compare at a glance

| Option | How access is controlled | Public endpoint used? | Pick it when |
|--------|-------------------------|----------------------|--------------|
| Server-level IP rule | IP range, all databases | Yes | Whole company office IP |
| Database-level IP rule | IP range, one database | Yes | Partner needs one database |
| Service endpoint + VNet rule | Subnet | **Yes** | Simple subnet-only access, public endpoint is OK |
| Private Link | Private IP in your VNet | **No** (can disable public access) | Highest security, no public exposure, cross-region, on-premises via VPN/ExpressRoute |
| VPN Gateway | Encrypted tunnel over internet | – | Low-cost hybrid link |
| ExpressRoute | Private dedicated circuit | – | High-bandwidth, compliance hybrid link |

## Remember for the exam
- Check order: **database rule → server rule → block**.
- Database-level rules: **T-SQL only**, in the user database. Server-level: portal/CLI/PowerShell or T-SQL in **master**.
- **256** rules per level. Managed Instance uses **NSGs**, not these IP rules.
- "Allow Azure services" = `0.0.0.0` rule, opens to **all** Azure customers. Avoid it.
- **Service endpoint** = subnet-based, still a public endpoint, one region.
- **Private Link** = private IP, no public internet, best protection; disable public access to force it.
- Redirect needs ports **11000–11999**; Proxy uses **1433**.
- SQL injection → **parameterized queries**; Defender for SQL detects attempts.

## Quick check
1. An external auditor needs access to only one of your 20 databases. What rule do you create, and where?
   <details><summary>Answer</summary>A database-level firewall rule with sp_set_database_firewall_rule, run inside that database.</details>
2. Security says the database must have no public endpoint at all. What do you use?
   <details><summary>Answer</summary>A private endpoint (Private Link), and set Public network access to Disable.</details>
3. Clients inside Azure fail to connect using the Redirect policy. Which ports should you check?
   <details><summary>Answer</summary>Outbound ports 11000–11999 (plus 1433 for the gateway). Redirect connects straight to the node on those ports.</details>
