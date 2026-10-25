> ⚠️ **Retired tool (Oct 2026 note):** Data Migration Assistant (DMA) was retired on **16 Jul 2025**. Learn what it did (exam questions may still mention it), but today use the **Azure Arc migration assessment** / **Azure DMS**. See [6 - [NEW] Troubleshoot a migration](<../Migrate SQL Server workloads to Azure SQL/6 - [NEW] Troubleshoot a migration.md>).

## ✅ **What Is Data Migration Assistant (DMA)?**

- DMA is a **free Microsoft tool** that helps assess and migrate **SQL Server databases**.
- It is mostly used when migrating from **on-premises SQL Server** to:

  - **SQL Server on Azure Virtual Machine**
  - **Newer on-premises SQL Server versions**

---

## 🔍 **Key Uses of DMA**

| Purpose                              | Description                                                                                                   |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| **Compatibility Check**              | Finds any **breaking changes, deprecated features, or unsupported functions** in your database.               |
| **Migration of Database Components** | Helps migrate **schema, data, users, server roles, and logins** to a SQL Server (on Azure VM or on-premises). |
| **Upgrade Readiness**                | Identifies **potential blockers** or things that must be fixed before upgrading SQL Server.                   |

---

## 🔑 **When to Use DMA vs Azure Database Migration Service (DMS)?**

- **DMA:** Small-scale migrations or assessments.
- **DMS:** Recommended for **large migrations** (automated, scalable, and supports online migrations).

---

## 🔒 **Migrating Logins with DMA**

You can use DMA to migrate:

- SQL Server logins
- Windows logins

⚠️ Limitations:

- Doesn’t support logins based on:

  - Standalone certificates
  - Asymmetric keys
  - SQL credentials

- Special logins like `##MS...##` and `sa` are **excluded** automatically.

➡️ During migration:

- Server roles and permissions are moved.
- The default owner for new server roles becomes **`sa`**.
- **Permissions are mapped** to the new target only if the matching login exists.

---

## ⚙️ **Tuning the DMA Tool Behavior**

The tool can be fine-tuned by editing the **dma.exe.config** file:

- File locations:

  - GUI: `%ProgramFiles%\Microsoft Data Migration Assistant\dma.exe.config`
  - CLI: `%ProgramFiles%\Microsoft Data Migration Assistant\dmacmd.exe.config`

Two common settings:

1. **Parallel Database Migration**

   - Default: migrate 8 databases in parallel.
   - Configurable:

   ```xml
   <advisorGroup>
     <workflowSettings>
       <migration parallelDatabases="8" />
     </workflowSettings>
   </advisorGroup>
   ```

2. **Connection Timeout**

   - Default: 15 seconds.
   - Configurable:

   ```xml
   <appSettings>
     <add key="ConnectionTimeout" value="15" />
   </appSettings>
   ```

---

## 👍 **Best Practices for Using DMA**

- Don’t install/run DMA on your production SQL Server.
- Run **assessments during non-peak hours** to avoid performance impact.
- If possible, **test against a recent backup**, not directly on production.
- Run assessments separately for:

  - **Compatibility issues**
  - **New feature recommendations**

- During migration:

  - Use a **shared folder** accessible by both source and target SQL Servers to avoid extra file copies.
  - Ensure correct **folder permissions** to avoid failures.
  - Prefer **encrypted connections** to secure the migration traffic.

---

## 📥 **Summary**

- DMA is a useful tool to **prepare for SQL Server migrations** and transfer your databases and logins.
- Use it for **assessments and smaller migrations**, or as part of a manual migration plan.
- For large, enterprise-wide migrations → prefer **Azure Database Migration Service (DMS).**

---
