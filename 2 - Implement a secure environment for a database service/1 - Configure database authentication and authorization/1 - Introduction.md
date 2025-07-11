### **Context of the Module**

### **Key Concepts Explained**

#### ✅ **Authentication and Authorization in Azure SQL**

- Azure SQL Database and **Azure SQL Managed Instance (MI)** use **Microsoft Entra ID** (formerly Azure Active Directory), rather than **Windows Server Active Directory (AD)** used in on-prem SQL Server.

  - This is important because cloud environments require **cloud-native identity solutions**.

#### ✅ **Best Practices Covered**

- **Least Privilege Principle**: Only give users the **minimum permissions needed** for their job. This helps reduce the risk of accidental or malicious actions.
- **Granular Permissions**: Instead of using broad, built-in roles that may grant **too many privileges**, assign permissions specifically to database objects like tables, views, or procedures.

---

### **What You’ll Learn in This Module**

1. **Authentication Options in Azure SQL**

   - How users and apps connect securely (Microsoft Entra ID, SQL authentication, etc.)

2. **Security Principals**

   - The identities (users, groups, applications) that can access the database.

3. **Object Permissions**

   - How to grant or deny permissions at the object level (e.g., SELECT on a table, EXECUTE on a procedure).

4. **Troubleshooting Authentication and Authorization Failures**

   - How to identify and resolve login or permission errors.

---

### **Next Step in the Learning Path**

The next unit will focus specifically on **"Describe authentication and identities"**, where you'll dive deeper into the types of users and identities that can authenticate to Azure SQL.

---

### **Bottom Line Summary**

This training module prepares you to **secure Azure SQL databases**, configure authentication and authorization, and follow best practices like least privilege and object-level permissions—important for any cloud DBA role.

---
