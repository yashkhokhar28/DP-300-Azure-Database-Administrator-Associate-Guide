## ✅ **Basic Database Permissions**

All relational databases, including SQL Server and Azure SQL, support **four common permissions** for data access:

| Permission | What it allows                           |
| ---------- | ---------------------------------------- |
| **SELECT** | Read/view data from a table or view.     |
| **INSERT** | Add new rows to a table or view.         |
| **UPDATE** | Modify existing rows in a table or view. |
| **DELETE** | Remove rows from a table or view.        |

- These permissions can be controlled using SQL commands:

  - **GRANT** → Gives permission.
  - **DENY** → Blocks permission, **overrides GRANT**.
  - **REVOKE** → Removes the GRANT or DENY.

➡️ Example: If a user is **granted SELECT** but **also denied SELECT**, the **DENY takes precedence**, and the user cannot select data.

---

## ✅ **Advanced Object Permissions**

SQL Server provides other permissions for deeper control:

| Permission               | What it allows                              |
| ------------------------ | ------------------------------------------- |
| **CONTROL**              | Full access to the object (like ownership). |
| **REFERENCES**           | See foreign keys on the object.             |
| **TAKE OWNERSHIP**       | Take ownership of the object.               |
| **VIEW CHANGE TRACKING** | See change tracking settings.               |
| **VIEW DEFINITION**      | View the object’s code/definition.          |

These permissions apply to **tables, views, functions, and procedures**.

For **functions and stored procedures**, there are also:

| Permission  | What it allows                  |
| ----------- | ------------------------------- |
| **ALTER**   | Modify the object’s definition. |
| **EXECUTE** | Run the function or procedure.  |

---

## ✅ **EXECUTE AS**

The **`EXECUTE AS`** command temporarily changes the execution context:

```sql
EXECUTE AS USER = 'username';
```

- Useful when a procedure needs to run with higher privileges than the caller.
- Available in **SQL Server** and **Azure SQL Managed Instance**.

---

## ✅ **Ownership Chains**

### ➤ What is it?

If the **owner of a procedure and the tables it uses is the same**, SQL Server allows the procedure to access those tables **even if the caller doesn’t have permission on the tables**.
This simplifies permission management.

Example flow:

1. A user has permission to **execute a procedure**.
2. The procedure queries a table.
3. If the **procedure and table share the same owner**, the user doesn’t need direct permission on the table.

➡️ This only works for **static SQL**, not dynamic SQL.

---

### ➤ **Example Problem**

If a user tries to directly query a table they don’t have permissions for:

```sql
SELECT * FROM Production.Product
```

➡️ They get an error.

But if they execute a **stored procedure** that queries the same table, it succeeds because of the **ownership chain**, **IF the owner of both objects is the same**.

---

### ➤ **Dynamic SQL breaks the chain**

If the stored procedure runs a **dynamic query** (using `sp_executesql`), ownership chaining doesn't apply.
The permissions of the **user running the dynamic query** are checked instead.

Example of dynamic SQL:

```sql
DECLARE @sql NVARCHAR(MAX)
SET @sql = 'SELECT * FROM Production.Product'
EXEC sp_executesql @sql
```

In this case, the user will get a permission error if they don’t have SELECT on `Production.Product`.

---

## ✅ **Least Privilege Principle**

- Give **only the permissions needed** to perform a task.
- Example: If an app only runs stored procedures, grant it only **EXECUTE**, not SELECT on tables.
- This reduces the chance of **accidental or malicious misuse**.

---

## ✅ **Dynamic SQL Summary**

- **Dynamic SQL** builds SQL statements at runtime.
- Example:

```sql
SELECT 'BACKUP DATABASE ' + name + ' TO DISK=''...' FROM sys.databases
```

- Executed using `sp_executesql`.
- You need **explicit permissions** on any object queried by dynamic SQL, because ownership chaining doesn't protect dynamic queries.

---

## ✅ **Key Takeaways**

| Concept               | Summary                                                                             |
| --------------------- | ----------------------------------------------------------------------------------- |
| Permissions           | Control access to data and objects.                                                 |
| GRANT / DENY / REVOKE | Grant = allow, Deny = block, Revoke = remove permission.                            |
| Ownership Chaining    | Allows stored procedures to access objects without giving users direct permissions. |
| Dynamic SQL           | Breaks ownership chaining, requires direct permissions.                             |
| Least Privilege       | Give only the minimum permissions needed for a task.                                |
| EXECUTE AS            | Temporarily switch execution to another user context.                               |

---
