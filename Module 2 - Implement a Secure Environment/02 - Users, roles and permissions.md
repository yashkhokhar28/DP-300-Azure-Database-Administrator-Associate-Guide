# Users, roles and permissions

> **Exam skills:** Configure security principals, Create users from Microsoft Entra identities, Configure database and object-level permissions using graphical tools, Apply the principle of least privilege for all securables, Manage authentication and authorization by using T-SQL
> **Reading time:** ~13 min

## The big idea
Once someone is logged in, SQL Server decides what they can touch. It does this with **principals** (who), **securables** (what), and **permissions** (allowed actions). Think of a hotel. The guest is the principal. The rooms are securables. The key card settings are permissions.

## Principals, securables and schemas
- **Security principal:** a person, group, app or process that can ask for access and be given permissions.
- **Securable:** a thing protected by permissions. Examples: a database, schema, table, view, stored procedure.
- **Schema:** a named folder inside a database that groups objects. Example: `Sales.Customers` is the `Customers` table in the `Sales` schema.
  - Every user has a **default schema**, usually `dbo`.
  - Best practice: always write the schema name, e.g. `SELECT Name FROM Sales.Customers;`.

Principals live at two levels:

| Server level (whole instance) | Database level (one database) |
|-------------------------------|-------------------------------|
| Logins, server roles | Users, database roles, application roles |

## Logins and users
- A **login** lets you into the **server**. It is stored in the `master` database.
- A **user** lets you into **one database**. A user is usually mapped to a login.

**Example:** A login is the key to the front door of a building. A user is the key to one flat inside it.

```sql
USE master;
CREATE LOGIN demo WITH PASSWORD = 'Pa55.w.rd';
USE WideWorldImporters;
CREATE USER demo FROM LOGIN demo;
```

### Contained users
A **contained user** lives only inside the database. It has **no login** in `master`. This is the **recommended** way in Azure SQL Database. The database is easier to move, because its users travel with it.

## Create users from Microsoft Entra identities
Use `FROM EXTERNAL PROVIDER` to point at an Entra user, group or app.

```sql
-- Contained database user for an Entra user (recommended in Azure SQL Database)
CREATE USER [dba@contoso.com] FROM EXTERNAL PROVIDER;

-- Server-level Entra login (in master)
CREATE LOGIN [nativeuser@contoso.com] FROM EXTERNAL PROVIDER;
```

- Entra **logins** (`CREATE LOGIN ... FROM EXTERNAL PROVIDER`) are available in Azure SQL Database (virtual `master`), Managed Instance, and SQL Server 2022+.
- You must be connected as an **Entra** account (for example the Entra admin) to create Entra users.

**Example:** A shop's web app uses a managed identity called `shop-web`. The DBA runs `CREATE USER [shop-web] FROM EXTERNAL PROVIDER;` and then gives it read rights. No password is needed.

## Roles: give permissions to groups, not people
A **role** is a group of principals. You grant permissions to the role once. Then you add or remove members.

```sql
CREATE ROLE SalesReader;
ALTER ROLE SalesReader ADD MEMBER DP300User1;
ALTER ROLE SalesReader ADD MEMBER DP300User2;
GRANT SELECT, EXECUTE ON SCHEMA::Sales TO SalesReader;
```

Now both users can read tables and run procedures in the `Sales` schema.

### Built-in database roles

| Role | What it allows |
|------|----------------|
| db_owner | Everything in the database |
| db_datareader | SELECT on all tables and views |
| db_datawriter | INSERT, UPDATE, DELETE on all tables and views |
| db_ddladmin | Create and change objects |
| db_securityadmin | Manage permissions (powerful, be careful) |
| db_accessadmin | Add or remove database users |
| db_backupoperator | Back up the database (SQL Server and MI; grants nothing in Azure SQL Database) |
| db_denydatareader / db_denydatawriter | Block all reads / all writes |
| public | Every user is in it; minimal rights |

### Application roles
An **application role** is turned on by an **app**, with a password. The app gets the role's permissions only for that session.

### Server roles on SQL Server and Managed Instance

| Role | What it allows |
|------|----------------|
| sysadmin | Everything on the server |
| serveradmin | Server settings, shut down |
| securityadmin | Manage logins and their permissions |
| processadmin | Kill running sessions |
| setupadmin | Manage linked servers |
| bulkadmin | Run BULK INSERT |
| diskadmin | Manage backup devices |
| dbcreator | Create, restore and drop databases |
| public | Every login; minimal rights |

### Server roles in Azure SQL Database
Azure SQL Database has **no** sysadmin you can use. It has **seven fixed `##MS_...##` server roles** (also in SQL Server 2022+). You **cannot** create custom server roles.

| Role | What it allows |
|------|----------------|
| ##MS_DatabaseConnector## | Connect to databases |
| ##MS_DatabaseManager## | Create and delete databases |
| ##MS_LoginManager## | Create and delete logins |
| ##MS_SecurityDefinitionReader## | Read security definitions |
| ##MS_DefinitionReader## | Read object definitions |
| ##MS_ServerStateReader## | View server state (e.g. DMVs) |
| ##MS_ServerStateManager## | Manage server state |

- Add members with T-SQL in the virtual `master`: `ALTER SERVER ROLE ##MS_ServerStateReader## ADD MEMBER [login];`.
- Not through the portal or CLI.
- Only the **server admin** or **Entra admin** can change membership.
- Members must be **logins** (SQL or Entra). Entra **groups are not supported** here.
- The older `master` database roles **loginmanager** (create logins) and **dbmanager** (create databases) still exist. Microsoft now recommends **##MS_LoginManager##** and **##MS_DatabaseManager##** instead.

## Permissions
The four everyday permissions on tables and views:

| Permission | Lets you |
|------------|----------|
| SELECT | Read rows |
| INSERT | Add rows |
| UPDATE | Change rows |
| DELETE | Remove rows |

More permissions:
- **CONTROL:** full rights on the object, like an owner.
- **ALTER:** change the object's definition.
- **EXECUTE:** run a procedure or function.
- **REFERENCES:** needed to create a foreign key that points to the table.
- **TAKE OWNERSHIP:** become the owner.
- **VIEW DEFINITION:** see the object's code.
- **VIEW CHANGE TRACKING:** read change tracking data (lesson 7).

### GRANT, DENY, REVOKE
- **GRANT** gives a permission.
- **DENY** blocks a permission. **DENY always wins** over GRANT.
- **REVOKE** removes an earlier GRANT or DENY (back to neutral).

**Example:** Ravi is in a role that has SELECT on `Sales.Orders`. The DBA also runs `DENY SELECT ON Sales.Orders TO Ravi;`. Ravi cannot read the table. The DENY beats the role's GRANT.

```sql
GRANT SELECT ON SCHEMA::Sales TO SalesReader;
DENY UPDATE ON Sales.Customers TO Intern1;
REVOKE UPDATE ON Sales.Customers FROM Intern1;  -- clears the deny
```

## Permissions with graphical tools
In **SSMS**, right-click a table, view or procedure → **Properties → Permissions**. Pick a user or role, then tick **Grant** or **Deny** next to each permission. You can do the same on the database (**Properties → Permissions**) or on a user (**Properties → Securables**). The result is the same as the T-SQL commands.

**Example:** In a CRM database, the DBA opens the `CustomerView` permissions in SSMS. She ticks Grant for SELECT and Deny for UPDATE for the "SalesTeam" role. Sales staff can read but not change customers.

## EXECUTE AS
`EXECUTE AS` runs code as a different user for a while. Use `REVERT` to go back.

```sql
EXECUTE AS USER = 'Tenant1';
SELECT * FROM Sales;   -- runs with Tenant1's rights
REVERT;
```

- Handy for testing what a user can see.
- `EXECUTE AS USER` works in SQL Server, Azure SQL Database and Managed Instance.
- `EXECUTE AS LOGIN` works only in **SQL Server** and **Managed Instance**.

## Ownership chaining
If a stored procedure and the tables it reads have the **same owner**, a user only needs **EXECUTE** on the procedure. They do not need rights on the tables.

**Example:** A user has EXECUTE on `dbo.GetProducts`. The procedure reads `Production.Product`. Both are owned by `dbo`. The user runs the procedure and it works. If the user runs `SELECT * FROM Production.Product` directly, it fails.

**Dynamic SQL breaks the chain.** Dynamic SQL is SQL text built and run at run time, often with `sp_executesql`. Then SQL Server checks the **caller's** own permissions on the tables.

```sql
DECLARE @sql NVARCHAR(MAX) = N'SELECT * FROM Production.Product';
EXEC sp_executesql @sql;   -- caller needs SELECT on the table
```

## Credentials (T-SQL)
- **Server credential** (`CREATE CREDENTIAL`): a stored secret used to reach outside resources. Can be used by many databases.
- **Database scoped credential** (`CREATE DATABASE SCOPED CREDENTIAL`): stored inside one database. It moves with the database. It is not mapped to a login or user.

**Example:** An ETL database reads files from Azure Storage. A database scoped credential holds the storage key, so the database can move to another server without redoing server setup.

## Least privilege
**Least privilege** means: give only the permissions needed for the job. Nothing more.

- Prefer **small, exact permissions** over big roles like `db_owner`.
- If an app only runs procedures, grant **EXECUTE** only, not SELECT on tables.
- Use **roles** and **groups**, not one-off grants to people.
- Keep Azure RBAC (resource management) apart from database permissions (data access).
- **Review** permissions regularly. Defender for SQL's vulnerability assessment can flag too many rights (lesson 6).

**Example:** A logistics company gives its reporting app only SELECT on the shipment tables. Later the app is hacked. The attacker can read shipments but cannot delete anything.

## Remember for the exam
- Login = server access (in `master`). User = database access.
- Azure SQL Database: prefer **contained users** and `CREATE USER ... FROM EXTERNAL PROVIDER`.
- **DENY beats GRANT**. REVOKE just removes a grant or deny.
- Grant to **roles**, then add members.
- Azure SQL Database has **7 fixed `##MS_...##` server roles**, managed only with T-SQL in `master`. No custom server roles. No Entra groups as members.
- `db_backupoperator` gives nothing in Azure SQL Database.
- Ownership chaining works for **static** SQL only. **Dynamic SQL** needs direct permissions.
- `EXECUTE AS LOGIN` does **not** work in Azure SQL Database.
- App only runs procedures → grant **EXECUTE** only.

## Quick check
1. A user has SELECT through a role and DENY SELECT given directly. Can they read the table?
   <details><summary>Answer</summary>No. DENY always overrides GRANT.</details>
2. You need a user to view DMVs at server level in Azure SQL Database. What do you do?
   <details><summary>Answer</summary>Add their login to ##MS_ServerStateReader## with ALTER SERVER ROLE in the virtual master database.</details>
3. A procedure uses sp_executesql to read a table. Users with EXECUTE get a permission error. Why?
   <details><summary>Answer</summary>Dynamic SQL breaks the ownership chain, so the caller needs SELECT on the table.</details>
