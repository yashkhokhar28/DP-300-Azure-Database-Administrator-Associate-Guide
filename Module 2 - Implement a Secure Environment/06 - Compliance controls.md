# Compliance controls

> **Exam skills:** Apply a data classification strategy, Configure server and database audits, Implement dynamic data masking, Implement ledger in Azure SQL, Implement row-level security
> **Reading time:** ~17 min

## The big idea
Laws like GDPR, HIPAA and PCI-DSS say you must know where sensitive data is, control who sees it, and prove what happened to it. Azure SQL gives you a toolbox:
- **Find and label** sensitive data → data classification, Microsoft Purview.
- **Hide** it → dynamic data masking, row-level security.
- **Record** who did what → auditing.
- **Prove** nobody tampered with it → ledger.
- **Watch for attacks** → Microsoft Defender for SQL.

Think of a pharmacy. Labels mark the dangerous medicines (classification). Staff see only their own shop's stock (row-level security). The till receipt hides most of the card number (masking). CCTV records who came in (auditing). A sealed, numbered logbook shows if a page was torn out (ledger). A security guard watches for thieves (Defender).

## Data classification
**Data classification** means tagging columns with labels that say how sensitive they are.
- **Sensitivity label:** how secret it is, e.g. Public, General, Confidential, Highly Confidential.
- **Information type:** what kind of data it is, e.g. Email, Credit Card, National ID.

**Example:** A healthcare app labels `PatientName` as Confidential and `MedicalHistory` as Highly Confidential. The team then knows which columns need encryption or masking.

### How to classify
- **Automatic recommendations:** in the Azure portal (database → **Security → Data Discovery & Classification**) or SSMS. The engine looks at **column names** like "email", "dob", "ssn". Badly named columns (like `col1`) will be missed.
- **Manual:** portal, SSMS, or T-SQL.

```sql
ADD SENSITIVITY CLASSIFICATION TO Application.People.EmailAddress
WITH (LABEL = 'Confidential', INFORMATION_TYPE = 'Email');
```

### Where labels are stored
- SQL Server 2019+, Azure SQL Database and Managed Instance: system view **`sys.sensitivity_classifications`**.
- Older SQL Server: **extended properties** on the column.

### Custom labels and policies
- Make your own labels and information types in **Microsoft Defender for Cloud** (the information protection policy). You need rights at the **root management group**.
- Or switch the tenant to the **Microsoft Purview Information Protection** policy to use Microsoft 365 sensitivity labels. This needs tenant-wide **Security Admin**, and automatic recommendations are turned off in that mode.
- Label-based **access control** for Azure SQL Database is **retired**. Labels are now for discovery, reporting and auditing only.

### Why it matters
- Shows where sensitive data lives.
- Helps you plan **encryption** and **masking**.
- **Auditing** can record when labelled columns are read (see below).

## Microsoft Purview
**Microsoft Purview** is a **data governance** service for the **whole organisation**, not just one database. It covers on-premises, multi-cloud (AWS, GCP) and SaaS sources.
- **Data map and catalog:** register sources, **scan** them, and let people **search** for data ("where are customer emails?").
- **Classification:** finds sensitive data automatically.
- **Lineage:** shows how data flows from source to destination (e.g. through stored procedures and pipelines).
- **Insights:** reports such as how much of the estate is scanned or holds sensitive data.

**Example:** A company with 10,000 databases uses Purview to find every column that holds credit card numbers, across Azure SQL, Power BI and on-premises servers.

### Scanning Azure SQL Database
1. **Register** the database in Purview.
2. Create a **scan** (which objects, which classification rules).
3. Test the connection and run it.

Authentication for the scan, best first:
- **System-assigned managed identity** of the Purview account (simplest, recommended).
- User-assigned managed identity.
- **Service principal**.
- **SQL authentication** (least recommended).

Network: allow Purview through the firewall (Allow Azure services), or use a **self-hosted integration runtime** or a **managed virtual network**. With a **self-hosted runtime, managed identities do not work**: use a service principal or SQL auth.

Stored-procedure lineage is in **preview**. The Purview identity needs **db_owner**, the database needs a master key, and "Allow Azure services" must be on. It does not work through a self-hosted or managed VNet runtime or a private endpoint.

### Classification vs Purview

| | Data Discovery & Classification | Microsoft Purview |
|--|------------------------------|-------------------|
| Scope | One database | Whole organisation, many source types |
| Extra features | Labels, audit of sensitive access | Catalog, search, lineage, insights |

## Auditing
**Auditing** records database events (logins, queries, changes) to a log. It helps with compliance and with spotting unusual activity.

### Where logs go

| Destination | Good for |
|-------------|----------|
| **Azure Storage account** | Cheap long-term archive |
| **Log Analytics workspace** | Queries and dashboards (Azure Monitor) |
| **Event Hubs** | Streaming to other systems (e.g. a SIEM) |

### Server-level vs database-level

| Type | Scope | When |
|------|-------|------|
| **Server-level** | All databases on the server, including new ones | **Recommended** default |
| **Database-level** | One database | Only if that database needs a different destination or different events |

- If server auditing is on, it **always applies**. A database audit runs **side by side** with it. It does **not** replace it.
- Avoid turning on both unless you have a reason (different storage, different rules).

**Example:** A bank turns on server-level auditing to Log Analytics for all 50 databases. Its fraud database must also keep logs in a separate storage account for regulators, so that one database also gets a database-level audit.

### Default events
The default action groups are:
- `BATCH_COMPLETED_GROUP` (every query and procedure run)
- `SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP`
- `FAILED_DATABASE_AUTHENTICATION_GROUP`

You can add more action groups.

### Good to know
- **Sensitive data access:** if columns are classified, the audit log field **`data_sensitivity_information`** shows which labelled columns a query touched.
- Turned on in the portal: **SQL server → Security → Auditing**.
- Read-only replicas are audited automatically.
- Under very heavy load, some events may be dropped.
- Storage retention default is **0 = keep forever**.
- Write to storage with a **managed identity** (recommended) or storage keys. Log Analytics and Event Hubs targets use a `SQLSecurityAuditEvents` diagnostic setting.
- Since July 2025, all server-level audit logs (including read-only replicas) go to one **`master`** folder.
- Failed **Entra** logins are **not** in the SQL audit log. Use the **Entra sign-in logs**.

## Dynamic data masking (DDM)
**DDM** hides part of a column's value **in query results**. The real data in the table is **not changed** and **not encrypted**.
- Users without the **UNMASK** permission see masked values.
- Users with UNMASK, or with **CONTROL** (db_owner, sysadmin), always see real data.
- No app changes needed.

**Example:** A call centre agent runs `SELECT Phone FROM Customers` and sees `XXX-XXX-1234`. The manager, who has UNMASK, sees the full number.

### Masking functions

| Function | Result | Example |
|----------|--------|---------|
| `default()` | Full mask: `XXXX`, `0`, `1900-01-01` | Address |
| `email()` | `aXXX@XXXX.com` | Email |
| `partial(prefix,"pad",suffix)` | Shows start and end, masks the middle | `partial(0,"XXXX-XXXX-",4)` for phone |
| `random(low,high)` | A random number in a range | `random(1,12)` |
| `datetime("Y")` | Masks part of a date (SQL Server 2022+) | Birth date |

The portal also offers a **credit card** mask (`XXXX-XXXX-XXXX-1234`).

```sql
ALTER TABLE dbo.Customer ALTER COLUMN Email
ADD MASKED WITH (FUNCTION = 'email()');
GRANT UNMASK TO Manager1;
GRANT UNMASK ON SCHEMA::Data TO ServiceManager;            -- granular (SQL Server 2022+, Azure SQL)
GRANT UNMASK ON Data.Membership(FirstName) TO Attendant;   -- one column
```

- Portal: database → **Security → Dynamic Data Masking**.
- Adding a mask needs **ALTER ANY MASK** plus ALTER on the table.
- **Copying keeps the mask:** `SELECT * INTO Copy FROM Customers` run by a user without UNMASK stores **masked** values.
- **Weakness:** clever users can guess values with queries (**inference**, e.g. `WHERE Salary > 50000`). Combine DDM with auditing, encryption and RLS.

## Row-level security (RLS)
**RLS** controls **which rows** a user can see or change. It works like an automatic `WHERE` clause added to every query by the database itself.

**Example:** A SaaS sales app stores all tenants in one `Sales` table. When Tenant1 queries it, they see only rows where `TenantName = 'Tenant1'`. They cannot see Tenant2's orders, however they write the query.

### How it works
1. Create an **inline table-valued function** (the **predicate**) with the rule.
2. Create a **security policy** that attaches the function to the table.

```sql
CREATE FUNCTION sec.fn_TenantFilter(@TenantName NVARCHAR(10))
RETURNS TABLE WITH SCHEMABINDING
AS RETURN SELECT 1 AS ok
WHERE @TenantName = USER_NAME() OR USER_NAME() = 'TenantAdmin';
GO
CREATE SECURITY POLICY sec.SalesPolicy
ADD FILTER PREDICATE sec.fn_TenantFilter(TenantName) ON dbo.Sales
WITH (STATE = ON);
```

Test with `EXECUTE AS USER = 'Tenant1'; SELECT * FROM dbo.Sales; REVERT;`.

### Two kinds of predicate

| Predicate | What it does | Affects |
|-----------|--------------|---------|
| **Filter** | Silently hides rows the user may not see | SELECT, UPDATE, DELETE |
| **Block** | Stops writes that break the rule | INSERT, UPDATE, DELETE |

### Tips
- RLS does **not encrypt** anything.
- Watch for **side-channel attacks** (e.g. divide-by-zero errors that reveal a hidden row exists).
- Put functions and policies in a **separate schema**. Keep predicate functions simple and fast.

## Ledger
**Ledger** makes data **tamper-evident**. It proves to auditors or partners that rows were not secretly changed. It is like blockchain, but inside one database, with no network of computers.
- Available in **Azure SQL Database, Managed Instance and SQL Server 2022+**.
- Each transaction is hashed with **SHA-256**. The hashes are **chained** together. If someone changes data outside normal SQL (e.g. edits the files), the chain breaks and **verification fails**.
- Ledger **detects** tampering. It does **not prevent** it.

### Two ledger table types

| Type | Allows | Use for |
|------|--------|---------|
| **Updatable ledger table** | Insert, update, delete. Old versions kept in a **history table** | Systems of record, e.g. bank balances |
| **Append-only ledger table** | **Insert only**. Updates and deletes are blocked | Logs, accounting journals, security events |

```sql
CREATE TABLE dbo.Balances (ID INT PRIMARY KEY, Amount MONEY)
WITH (SYSTEM_VERSIONING = ON, LEDGER = ON);           -- updatable
CREATE TABLE dbo.AuditEvents (ID INT, Msg NVARCHAR(200))
WITH (LEDGER = ON (APPEND_ONLY = ON));                -- append-only
```

### Ledger database and digests
- A **ledger database** makes **every** table a ledger table. You choose this **only when creating** the database (portal → **Security** tab → Ledger → **Enable for all future tables**). You **cannot** turn it on or off later.
- You can still make single ledger tables in a normal database with T-SQL.
- A **digest** is the latest hash of the chain. Store digests outside the database: **Azure Blob Storage with an immutability policy** or **Azure Confidential Ledger** (automatic), or save them manually.
- Verify with `sys.sp_verify_database_ledger_from_digest_storage` (or `sys.sp_verify_database_ledger` with a saved digest).

**Example:** A fintech company keeps loan approvals in an updatable ledger table and sends digests to immutable Blob Storage. An auditor runs verification and sees the chain is intact, so no approval was quietly edited.

## Microsoft Defender for SQL
**Defender for SQL** is a security service. It covers Azure SQL Database, Managed Instance, SQL Server on Azure VMs, SQL Server enabled by **Azure Arc** (on-premises or other clouds), and Synapse dedicated SQL pools. It has two parts.

### 1. Vulnerability assessment (VA)
Scans for **weak settings**: too many permissions, weak authentication, unprotected sensitive data, bad configuration. It gives fix-it recommendations.
- **Express configuration** (recommended): results stored by Microsoft, **no storage account** needed.
- **Classic configuration**: results go to **your storage account** (classic VA/ATP APIs retire 16 Aug 2027).

**Example:** VA flags that the reporting app's user is in `db_owner` but only needs to read. You move it to `db_datareader`.

### 2. Advanced Threat Protection (ATP)
Watches activity and raises **real-time alerts**:
- **Vulnerability to SQL injection** (a badly written query that could be exploited).
- **Potential SQL injection** (an actual attack attempt).
- Access from an **unusual location**, an **unfamiliar Azure data centre**, or an **unfamiliar principal**.
- Access from a **potentially harmful application** (known attack tools).
- **Brute force** SQL credentials (many failed passwords).

**Example:** 100 failed logins arrive from an unknown country in two minutes. Defender raises a brute-force alert.

### Turning it on
- Per server or instance: **Microsoft Defender for Cloud** section → set Defender for SQL **On**.
- Or for the whole **subscription**: Defender for Cloud → **Environment settings → Defender plans → Databases**. Plans: **Defender for Azure SQL Databases** (SQL DB, MI, Synapse) and **Defender for SQL Servers on Machines** (VMs, Arc). New resources are covered automatically.
- Turn on **auditing** too, so you can investigate alerts.
- Who can configure it: **SQL Security Manager** or a server/database admin.

## Compare at a glance

| Need | Feature |
|------|---------|
| Find and label sensitive columns in a database | Data Discovery & Classification |
| Find sensitive data and lineage across the company | Microsoft Purview |
| Record who logged in and what they ran | Auditing |
| Show only part of a value to some users | Dynamic data masking |
| Each user sees only their own rows | Row-level security |
| Prove data was not tampered with | Ledger |
| Find weak settings / get attack alerts | Defender for SQL (VA / ATP) |
| Hide values even from DBAs | Always Encrypted (lesson 4) |

## Remember for the exam
- Classification labels are in **`sys.sensitivity_classifications`**; T-SQL `ADD SENSITIVITY CLASSIFICATION`. Auto-recommendations rely on **column names**.
- Audit to **Storage** (archive), **Log Analytics** (query), **Event Hubs** (stream). Prefer **server-level**; database audit runs **alongside**, not instead.
- `data_sensitivity_information` in audit logs shows access to classified columns.
- DDM changes **output only**; data is not encrypted. **UNMASK** reveals; db_owner always sees real data. Masks survive `SELECT INTO` for non-UNMASK users.
- RLS = inline TVF + **security policy**. **Filter** hides rows; **block** stops writes.
- Ledger = **tamper-evidence**, SHA-256 chain. Updatable vs **append-only**. Ledger **database** only at creation. Digests to **immutable Blob** or **Azure Confidential Ledger**.
- Defender for SQL = **VA** (misconfigurations) + **ATP** (SQL injection, brute force, unusual access).
- Purview scan with a **self-hosted runtime** → no managed identity; use a service principal or SQL auth.

## Quick check
1. Call centre staff must see only the last four digits of card numbers; managers must see the full number. What do you use?
   <details><summary>Answer</summary>Dynamic data masking with a partial() (or credit card) mask, and GRANT UNMASK to managers.</details>
2. Regulators need proof that an accounting journal was never edited or deleted. Which table type?
   <details><summary>Answer</summary>An append-only ledger table. It allows inserts only and its hash chain shows any tampering.</details>
3. You want the same audit settings on every current and future database on a server. Where do you configure auditing?
   <details><summary>Answer</summary>At the server level. It applies to all databases on the server.</details>
