# Authentication

> **Exam skills:** Configure Microsoft Entra ID authentication for Azure SQL Database, Azure SQL Managed Instance, and SQL Server, Configure authentication for SQL on Azure VMs and Azure SQL Managed Instance
> **Reading time:** ~10 min

## The big idea
**Authentication** answers "Who are you?". **Authorization** answers "What are you allowed to do?". This lesson is about the first question. Think of an office building: the guard at the door checks your ID card (authentication). Your key card then opens only some rooms (authorization).

## The two main ways to log in
SQL Server and Azure SQL support two main kinds of login.

- **SQL authentication.** A username and password stored **inside** SQL Server. Easy, but less secure. Each password must be managed by hand.
- **Microsoft Entra ID authentication** (Entra ID was called Azure Active Directory, or Azure AD, before). Users log in with their company cloud account. This is the same account they use for the Azure portal and Microsoft 365.
- **Windows authentication.** Used by on-premises SQL Server and SQL Server on Azure VMs. Users log in with their Windows (Active Directory) account. A VM can join the company Active Directory for single sign-on.

Why Entra ID is better:
- One place to manage people. If you **disable** a user in Entra ID, they lose database access at once.
- It supports **MFA** (multi-factor authentication: a second check like a phone push, SMS, or one-time code).
- It supports **single sign-on** (log in once, use many services) and **Conditional Access** (rules like "only from company laptops").

**Example:** A shop worker leaves the company. With SQL logins, the DBA must find and drop the login on every server. With Entra ID, HR disables one account and all database access stops.

## Which option works where

| Platform | SQL auth | Entra ID auth | Classic Windows auth |
|----------|----------|---------------|--------------|
| SQL Server on Azure VM | Yes | Yes (SQL Server 2022+) | Yes (VM joined to Active Directory) |
| Azure SQL Managed Instance | Yes | Yes | No domain join |
| Azure SQL Database | Yes | Yes | No |

Note: Azure SQL Database and Managed Instance are cloud services. They use **Entra ID** in place of the on-premises Windows Active Directory. On a VM, you control everything (it is your own SQL Server), so all three options are open to you.

## Set up Entra ID authentication on Azure SQL Database and Managed Instance
The key step is to set a **Microsoft Entra admin** for the server.

- Set it in the **Azure portal** (logical server → **Settings → Microsoft Entra ID** → set admin → Save), or with **PowerShell** or **Azure CLI**.
- This setting is **server-wide**, not per database.
- The Entra admin gets **sysadmin-like** power on all databases on the server.
- **Best practice:** make an Entra **group** the admin, not one person. If a person leaves, the group still works.

**Example:** A school creates an Entra group called "SQL-Admins" with three DBAs. The group is set as the Entra admin. When one DBA leaves, they are just removed from the group.

### Microsoft Entra-only authentication
You can switch off SQL authentication completely. This is called **Microsoft Entra-only authentication**.

- Works on Azure SQL Database and Managed Instance.
- It also blocks the **SQL server admin** login (the one made with a password when the server was created).
- You must set the **Entra admin first**.
- Turn it on in the portal, or with:
  - `az sql server ad-only-auth enable` / `Enable-AzSqlServerActiveDirectoryOnlyAuthentication`
  - Managed Instance: `az sql mi ad-only-auth enable`
- You can also force it across many servers with **Azure Policy**.
- Who can switch it on or off: **SQL Security Manager**, **Owner**, or **Contributor**. **SQL Server Contributor cannot** (this keeps duties separate).

### Connecting with MFA from SSMS
In SQL Server Management Studio (SSMS), pick the authentication option **Microsoft Entra MFA**. Its old name was "Universal with MFA".

## Entra ID on SQL Server (VM or on-premises)
SQL Server 2022 and later can also use Entra ID. In short:
- Register a Microsoft Entra application and give it permissions.
- Create and assign a certificate.
- Set it up in the Azure portal, then create logins and users.
- Connect with SSMS using an Entra option.

You do not need to remember every click. Know that **SQL Server 2022+** supports Entra ID logins.

## Identities for apps: service principals and managed identities
People are not the only ones who log in. Apps and Azure services log in too.

- **Service principal.** An identity for an app, registered in Entra ID. It proves itself with a **secret** (like a password) or a **certificate**. You must store and rotate that secret.
- **Managed identity.** An identity that Azure creates and manages for an Azure resource (a VM, a web app, a SQL server). **No password** is stored anywhere. Azure handles the secret for you.
  - **System-assigned:** tied to one resource. It is deleted when the resource is deleted.
  - **User-assigned:** a separate Azure resource. You can give it to many resources, and it lives on its own.

**Example:** A web app reads orders from Azure SQL Database. With SQL auth, the password sits in a config file. With a managed identity, the app has no password at all. The DBA just creates a database user for the app's identity.

Where managed identities show up in this module:
- An app connects to Azure SQL with no password.
- A SQL server reads its **TDE key** from Azure Key Vault (lesson 4).
- Auditing writes to a storage account (lesson 6).
- Microsoft Purview scans a database (lesson 6).

## Azure RBAC: managing the resource, not the data
**Azure RBAC** (role-based access control) controls what someone can do to the **Azure resource**. For example: create, delete, or scale a database. It does **not** let them read the data inside.

| Role | Can do | Cannot do |
|------|--------|-----------|
| SQL DB Contributor | Create and manage databases | Read data |
| SQL Server Contributor | Create, delete and configure servers and databases | Read data |
| SQL Security Manager | Manage security settings (auditing, threat detection, Entra-only auth) | Read data |

**Example:** A DevOps engineer has SQL DB Contributor. She can deploy a new database for the payroll team. She still cannot run `SELECT` on the salary table. That needs a **database** permission.

This is called **separation of duties**: people who run the infrastructure are not automatically allowed to see the data.

## Compare at a glance

| Method | Password stored? | Best for |
|--------|------------------|----------|
| SQL authentication | Yes, in SQL Server | Old apps, quick tests |
| Entra ID user or group | No (Entra handles it) | People; supports MFA |
| Service principal | Yes, a secret or certificate | Apps outside Azure, automation |
| Managed identity | No | Azure resources talking to Azure SQL or Key Vault |
| Windows authentication | No | SQL Server on a domain-joined VM |

## Remember for the exam
- Authentication = who you are. Authorization = what you can do.
- Best practice: **Entra ID + MFA**, and an Entra **group** as the server admin.
- The Entra admin is set **per server**, not per database.
- **Entra-only authentication** turns off all SQL logins, including the SQL server admin. Set the Entra admin first.
- **SQL Server Contributor cannot** turn Entra-only auth on or off. SQL Security Manager can.
- Azure RBAC roles manage resources; they **never** give data access.
- "App in Azure, no passwords" → **managed identity**.
- SSMS option for MFA = **Microsoft Entra MFA** (old name: Universal with MFA).

## Quick check
1. A company wants to block all password-based logins to its Azure SQL Database, including the server admin. What should it enable?
   <details><summary>Answer</summary>Microsoft Entra-only authentication. It disables SQL authentication completely, but an Entra admin must be set first.</details>
2. A user has the SQL DB Contributor role. Can they read customer rows?
   <details><summary>Answer</summary>No. Azure RBAC roles manage the resource only. Reading data needs database permissions.</details>
3. A web app in Azure must connect to Azure SQL Database with no stored password. What should you use?
   <details><summary>Answer</summary>A managed identity. Azure manages its credentials, so nothing is stored in code or config.</details>
