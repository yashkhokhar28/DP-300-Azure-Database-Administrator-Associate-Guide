## ✅ **Authentication in SQL Server & Azure SQL**

### ➤ **On-Premises SQL Server & Azure VMs**

- **SQL Server Authentication**:

  - Username & password **stored inside SQL Server**.
  - Common but less secure.

- **Windows Authentication** (for on-prem) / **Microsoft Entra ID Authentication** (for Azure):

  - Users log in with their **Microsoft Entra ID (formerly Azure AD)** accounts.
  - More secure, **easier to manage user accounts**.
  - Example: Disable a user in Entra ID and they automatically lose database access.

---

### ➤ **Azure SQL Database & Managed Instance**

- Supports **SQL Server Authentication** and **Microsoft Entra Authentication**.
- Microsoft Entra Authentication lets users use the **same credentials as the Azure Portal, Microsoft 365, etc.**
- Entra ID supports:

  - **MFA (Multi-Factor Authentication):**

    - Push notification, SMS, or one-time access codes.

  - Requires **Universal Authentication** when connecting from SQL Server Management Studio (SSMS).

---

### ➤ **Admin Account Setup**

- In **Azure SQL Database**, admin permissions are set via the **Azure Portal**, **PowerShell**, or **Azure CLI**.
- Best practice: Assign a **Microsoft Entra Group** (instead of a single user) as the admin.

  - This avoids problems if a user leaves the company.

- The admin group gets **sysadmin-like permissions** across all databases on the server.
- This setting is **server-wide**, not at the database level.

---

## ✅ **Role-Based Access Control (RBAC)**

Azure uses **RBAC** to control what users can do **in the Azure Portal, CLI, or PowerShell**, **outside of the database itself**.

- Example:

  - Someone can create or delete an Azure SQL Database, but **cannot read data from it** unless they also have database-level permissions.

- RBAC is used to **separate duties**:

  - Deployment and management of resources vs. accessing the actual data.

### ➤ **Built-in RBAC Roles for Azure SQL:**

| Role Name              | What they can do                                            | What they cannot do |
| ---------------------- | ----------------------------------------------------------- | ------------------- |
| SQL DB Contributor     | Create and manage databases                                 | Cannot access data  |
| SQL Security Manager   | Manage security policies (auditing, threat detection, etc.) | Cannot access data  |
| SQL Server Contributor | Manage servers and databases (create, delete, configure)    | Cannot access data  |

Example use case:
A DevOps engineer can deploy a database but won't be able to view the sensitive data inside unless explicitly granted.

---

## ➤ **Key Takeaways**

- **Microsoft Entra ID Authentication + MFA = Best Practice for Security.**
- Assign **groups instead of individuals** for admin access.
- Use **RBAC roles for management tasks**, and database permissions for data access.
- Authentication (who you are) and authorization (what you can do) are separated cleanly.

---
