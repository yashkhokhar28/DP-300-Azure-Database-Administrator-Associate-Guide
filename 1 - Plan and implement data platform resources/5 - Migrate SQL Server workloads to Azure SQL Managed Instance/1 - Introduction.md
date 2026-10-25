## 🔑 **Key Idea: Easy Lift-and-Shift Migration**

**Azure SQL Managed Instance** is a **Platform as a Service (PaaS)** solution that allows you to:

- Migrate **existing SQL Server databases** from on-premises **with almost no code or schema changes.**
- Keep most SQL Server features that other Azure SQL options don’t support (e.g., SQL Agent, linked servers, CLR, etc.).
- Enjoy **cloud scalability, backups, and high availability**—without managing the underlying server OS or hardware.

---

## 🛍️ **Example Scenario**

You work for a **sports clothing retailer**.

- Your company is moving systems to Azure to reduce infrastructure management costs and improve uptime.
- But your **business-critical databases** haven’t moved yet because:

  - You think Azure SQL Database (the simpler PaaS offering) might require database rewrites or feature compromises.
  - You want to avoid spending a lot of **DBA/developer time** adapting the database.

➡️ **Solution:** Use **Azure SQL Managed Instance**, because it **fully supports most SQL Server features**, allowing an almost direct migration.

---

## 🚀 **What You’ll Learn in This Module**

This training module will teach you:

- What **Azure SQL Managed Instance** is.
- How it compares to **Azure SQL Database** (which is more restrictive) and SQL Server on Azure VMs (IaaS).
- How to pick the right Azure SQL service (**Managed Instance, Database, or VM**) for your workload.
- How to perform a migration efficiently.

---

## ⚙️ **Why Use Managed Instance?**

- ✅ Supports SQL Agent, linked servers, Service Broker, and other advanced SQL features.
- ✅ Simplifies **lift-and-shift** migrations.
- ✅ No need to redesign your database for PaaS.
- ✅ Built-in backup, patching, and high availability.
- ✅ Lower admin overhead than VMs running SQL Server (which are IaaS).

---

## 🆚 **How Managed Instance Differs from Other Azure SQL Options**

| Feature                             | Azure SQL Database  | Azure SQL Managed Instance | SQL Server on Azure VM |
| ----------------------------------- | ------------------- | -------------------------- | ---------------------- |
| Migration effort (for legacy SQL)   | High (many changes) | Low (lift and shift)       | Very low (re-host)     |
| SQL Agent support                   | ❌                  | ✅                         | ✅                     |
| Linked servers                      | ❌                  | ✅                         | ✅                     |
| Backups / Patching managed by Azure | ✅                  | ✅                         | ❌ (you manage it)     |
| OS-level access                     | ❌                  | ❌                         | ✅                     |
| Best for                            | Cloud-native apps   | Modernizing legacy apps    | Legacy re-hosting      |

---
