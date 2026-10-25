## ✅ **What Are Security Principals?**

Security principals are **people, applications, or processes** that:

- **Request access to SQL Server resources**
- **Have permissions** granted to them.

They exist at two levels:

| Server Level (whole instance) | Database Level (single database) |
| ----------------------------- | -------------------------------- |
| Logins, Server Roles          | Users, Database Roles            |

---

## ✅ **Schemas and Securables**

- **Securables:**
  These are **resources** protected by permissions.

  - Examples: **Databases, Tables, Views, Stored Procedures**.

- **Schema:**

  - A **namespace** for organizing objects within a database.
  - Example: `SalesSchema.Customers`
  - Every user has a **default schema**, normally `dbo` unless specified.
  - **Best practice:** Always mention the schema when querying:

    ```sql
    SELECT name FROM SalesSchema.Customers;
    ```

---

## ✅ **Logins and Users**

### ➤ **Logins** = Access to SQL Server Instance

- Created at the **server level** (stored in `master`).
- Types: SQL Authentication, Windows Authentication, or Microsoft Entra ID Authentication. Microsoft Entra logins (`CREATE LOGIN [name] FROM EXTERNAL PROVIDER`) are GA in Azure SQL Database (virtual `master`), Managed Instance and SQL Server 2022+. 🔄

### ➤ **Users** = Access to a Database

- Created inside **each database**, mapped to logins.
- In Azure SQL Database, **use contained users** (recommended). Example:

  ```sql
  CREATE USER [dba@contoso.com] FROM EXTERNAL PROVIDER;
  ```

  This creates a user linked to Microsoft Entra ID.

### ➤ **Example of Login + User Mapping**

```sql
USE master;
CREATE LOGIN demo WITH PASSWORD = 'Pa55.w.rd';

USE WideWorldImporters;
CREATE USER demo FROM LOGIN demo;
```

---

## ✅ **Database Roles**

- **Database roles = groups of users with shared permissions.**
- Users are added to a role; permissions are granted to the role.
- Example:

```sql
CREATE ROLE SalesReader;
ALTER ROLE SalesReader ADD MEMBER DP300User1;
ALTER ROLE SalesReader ADD MEMBER DP300User2;
GRANT SELECT, EXECUTE ON SCHEMA::Sales TO SalesReader;
```

Now both users can query and run procedures in the `Sales` schema.

---

## ✅ **Application Roles**

- Special roles activated by **an application** using a password.
- Apply permissions to the application during its session.

---

## ✅ **Built-in Database Roles**

| Database Role     | Purpose                                                       |
| ----------------- | ------------------------------------------------------------- |
| db_owner          | Full control over the database.                               |
| db_datareader     | Can SELECT from all tables/views.                             |
| db_datawriter     | Can INSERT/UPDATE/DELETE on all tables/views.                 |
| db_ddladmin       | Can create/modify database objects.                           |
| db_securityadmin  | Can manage permissions (caution: powerful).                   |
| db_backupoperator | Can back up the database (SQL Server & MI; grants nothing in Azure SQL DB). 🔄 |
| db_accessadmin    | Can add/remove database users.                                |
| db_denydatareader | Explicitly deny read access.                                  |
| db_denydatawriter | Explicitly deny write access.                                 |
| public            | All users belong to this by default, has minimal permissions. |

---

## ✅ **Azure SQL Special Admin Roles**

| Database Role | Purpose                                                     |
| ------------- | ----------------------------------------------------------- |
| loginmanager  | Can create new logins (like `securityadmin` on SQL Server). |
| dbmanager     | Can create and delete databases (like `dbcreator`).         |

- Microsoft now recommends the server roles **##MS_LoginManager##** and **##MS_DatabaseManager##** instead of these `master` database roles. 🔄

---

## ✅ **Fixed Server-Level Roles (SQL Server & Managed Instance)**

| Server Role   | Purpose                                              |
| ------------- | ---------------------------------------------------- |
| sysadmin      | All permissions on the server.                       |
| serveradmin   | Manage server-wide settings, shutdown server.        |
| securityadmin | Manage logins and grant/revoke permissions.          |
| processadmin  | Kill running SQL processes.                          |
| setupadmin    | Manage linked servers.                               |
| bulkadmin     | Run BULK INSERT commands.                            |
| diskadmin     | Manage backup devices.                               |
| dbcreator     | Create, restore, and drop databases.                 |
| public        | All logins belong to this role, minimal permissions. |

---

### ➤ Azure SQL Database fixed server-level roles (`##MS_...##`, also in SQL Server 2022+): 🔄

| Role Name                       | Purpose                               |
| ------------------------------- | ------------------------------------- |
| ##MS_DatabaseConnector##        | Allows connections to databases.      |
| ##MS_DatabaseManager##          | Manage databases.                     |
| ##MS_LoginManager##             | Manage logins.                        |
| ##MS_SecurityDefinitionReader## | Read security definitions.            |
| ##MS_ServerStateReader##        | View server state.                    |
| ##MS_ServerStateManager##       | Manage server state.                  |
| ##MS_DefinitionReader##         | Read definitions of database objects. |

- Add members with T-SQL in the virtual `master` database, e.g. `ALTER SERVER ROLE ##MS_ServerStateReader## ADD MEMBER [login];`. Only the **server admin** or **Microsoft Entra admin** can change membership. Members are **logins** (SQL or Entra); Entra **groups** aren't supported. 🔄

---

## ✅ **Key Takeaways**

- **Security Principals = Users, Logins, and Roles** that control access.
- Use **schemas** to organize objects.
- Grant permissions through **roles** for easier management.
- In Azure SQL Database:

  - Prefer **contained users** and Microsoft Entra ID for authentication.
  - Use built-in database roles or create custom roles for fine-grained security.
  - Server-level roles are **limited to the 7 fixed `##MS_...##` roles** (no custom server roles) and are managed with **T-SQL (`ALTER SERVER ROLE`) in `master`**, not with the portal or CLI. 🔄

---
