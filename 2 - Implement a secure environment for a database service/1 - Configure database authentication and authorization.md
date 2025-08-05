# ✅ **Mastering Authentication & Authorization in Azure SQL — Full Module Summary**

---

## 🔹 **Context & Purpose**

In the cloud world, securing your database isn't just about passwords anymore. This module equips you to manage **identity, access, and permissions** for **Azure SQL Databases** and **Managed Instances (MI)** using **Microsoft Entra ID** (formerly Azure AD), while following best practices like **least privilege** and **role-based access control (RBAC)**.

---

## 🔹 **Core Concepts**

### 🔐 **Authentication vs Authorization**

- **Authentication**: Verifies _who_ you are.
- **Authorization**: Controls _what_ you can do.

Azure SQL uses **Microsoft Entra ID**, not on-prem Windows AD, to handle identities — allowing for modern, scalable, and secure identity management.

---

## 🔹 **What You’ll Learn**

1. Auth methods in Azure SQL: SQL Auth, Entra ID, MFA, etc.
2. Understanding **Security Principals**: Users, groups, apps.
3. Assigning **object-level permissions** (e.g., SELECT, EXECUTE).
4. How to **troubleshoot login and access failures** like a pro.

---

## 🔹 **Best Practices Recap**

- **Least Privilege**: Never give more access than necessary.
- **Granular Permissions**: Prefer direct permissions over broad roles.
- **Use Groups** for admin access to avoid user-dependence.
- Leverage **RBAC** for managing Azure resources; use **SQL permissions** for data access.

---

## 🔹 **Authentication in Action**

### ▶️ **On-Prem SQL vs Azure SQL**

| Environment  | Authentication Option                          | Notes               |
| ------------ | ---------------------------------------------- | ------------------- |
| On-prem / VM | SQL Auth, Windows Auth                         | Traditional methods |
| Azure SQL    | SQL Auth, Entra ID Auth (+MFA, Universal Auth) | Modern & secure     |

### 🔑 **Entra ID Benefits**

- Single sign-on with Azure Portal, M365, etc.
- Supports **MFA** (push/SMS/OTP).
- Easy user revocation.
- More secure and manageable than SQL Auth.

---

## 🔹 **Admin Setup & Access Control**

- **Admin permissions** are set at the **server level**, often using **Entra Groups**.
- Assign roles like **dbmanager** or **loginmanager** to control DBs and logins.
- Always prefer **groups** over individual user accounts.

---

## 🔹 **RBAC in Azure**

- **Azure-level access** managed via **RBAC**, not SQL permissions.
- Example roles:

  - `SQL DB Contributor`: Can create DBs, not access data.
  - `SQL Security Manager`: Can manage policies, not query data.

👉 Separate management of infrastructure from data access — clean and secure.

---

## 🔹 **Security Principals Deep Dive**

| Level    | Entity Type   | Examples                   |
| -------- | ------------- | -------------------------- |
| Server   | Logins, Roles | `CREATE LOGIN`, `sysadmin` |
| Database | Users, Roles  | `CREATE USER`, `db_owner`  |

✅ Use **contained users** in Azure SQL for better portability and Entra ID integration.

---

## 🔹 **Schemas & Securables**

- **Securables**: Any object you can assign permissions to (tables, views, procs).
- **Schemas**: Organize securables like folders. Use them explicitly:

```sql
SELECT * FROM SalesSchema.Customers;
```

---

## 🔹 **Logins vs Users**

- **Login** = Access to the SQL server instance.
- **User** = Access inside a specific database.
- Use `CREATE LOGIN` + `CREATE USER` (or `CREATE USER FROM EXTERNAL PROVIDER` for Entra).

---

## 🔹 **Roles for Simplified Access Control**

### ✅ **Database Roles**

- Built-in roles like `db_datareader`, `db_datawriter`, `db_owner`, etc.
- Custom roles: Group users and assign them shared access.

```sql
CREATE ROLE SalesReader;
ALTER ROLE SalesReader ADD MEMBER SomeUser;
GRANT SELECT ON SCHEMA::Sales TO SalesReader;
```

### ✅ **Application Roles**

- Temporary, session-level roles used by applications with passwords.

---

## 🔹 **Server-Level Roles (SQL Server / Managed Instance)**

| Role          | Purpose                         |
| ------------- | ------------------------------- |
| sysadmin      | God mode                        |
| securityadmin | Manage logins/permissions       |
| dbcreator     | Create and drop DBs             |
| public        | Default for all, minimal access |

⚠️ Azure SQL DB has internal equivalents like:

- `##MS_DatabaseManager##`, `##MS_LoginManager##`, etc.

---

## 🔹 **Permissions Management 101**

| Command | Purpose                        |
| ------- | ------------------------------ |
| GRANT   | Allow access                   |
| DENY    | Block access (overrides GRANT) |
| REVOKE  | Remove permission (neutral)    |

Basic permissions:

- SELECT, INSERT, UPDATE, DELETE

Advanced:

- CONTROL, EXECUTE, ALTER, VIEW DEFINITION, etc.

---

## 🔹 **EXECUTE AS & Ownership Chains**

- Use `EXECUTE AS USER = 'name'` to temporarily escalate permissions.
- **Ownership Chaining**: If proc & tables share same owner, access is inherited.

  - ✅ Works for **static SQL**
  - ❌ Breaks for **dynamic SQL** (e.g., `sp_executesql`)

---

## 🔹 **Dynamic SQL & Security**

- Requires **explicit permissions**, as **ownership chaining is bypassed**.
- Example:

```sql
EXEC sp_executesql N'SELECT * FROM SomeTable';
```

If the user doesn’t have direct SELECT on `SomeTable`, it fails.

---

## 🔹 **Common Access Issues & Fixes**

### ❌ **Login Failed**

Check:

- Login exists? Enabled?
- Mapped to a user in DB?
- Has required permissions?

### 🧱 **Firewall / Network**

- Validate firewall settings on server and local machine.

### 🚧 **Resource Limits**

- Check storage, CPU, DTUs — might need to scale up.

---

## 🔹 **Transient Faults (Temporary Errors)**

Azure infra may reconfigure on the fly:

- Maintenance, auto-failover, or scaling events.
- Errors like:

  - “Cannot open database”
  - “Not enough resources”
  - “Too many operations”

✅ Usually resolve within **60 seconds**.

---

## 🔁 **Retry Logic (App Side)**

Use **exponential backoff** strategy:

```plaintext
5s → 10s → 20s → 40s → max 60s
```

- Always retry with **new connections**.
- Don't spam retries endlessly — max limit is your friend.

---

## 🔎 **Connection String Sanity Check**

Update your app’s connection string when:

- Changing server
- Adding a new DB
- Rotating credentials

From Azure Portal:
`SQL Database → Connection Strings → Copy and Update`

---

## 🚨 **Troubleshooting Login Issues: Quick Fix Flow**

1. Check login exists:

```sql
SELECT name FROM sys.sql_logins;
```

2. Enable if disabled:

```sql
ALTER LOGIN user ENABLE;
```

3. Map to DB:

```sql
CREATE USER user FOR LOGIN user;
```

4. Assign permissions:

```sql
GRANT SELECT ON SCHEMA::Sales TO user;
-- or add to db_datareader
```

---

## 💡 **Final Takeaways**

| Topic            | Pro Tip                                                         |
| ---------------- | --------------------------------------------------------------- |
| Auth Methods     | Prefer **Microsoft Entra ID + MFA** for cloud apps              |
| Permissions      | Use **roles** for easier and scalable access control            |
| Ownership Chains | Good for simplifying access – but **avoid dynamic SQL abuse**   |
| Troubleshooting  | Always check: login status, user mapping, permissions, firewall |
| Transient Errors | Normal in Azure – handle with retries, not panic                |

---

🎯 **Bottom Line**:
If you're securing Azure SQL like a boss, you **don’t just slap logins and hope for the best**. You **plan identity, permissions, and fallback**. You **grant the least**, automate retries, monitor usage, and sleep peacefully.

This module sets you up for exactly that. You’re now officially more dangerous (in a good way) with Azure SQL security. 💪

---
