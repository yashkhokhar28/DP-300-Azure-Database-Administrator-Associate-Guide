## ✅ **What is Azure SQL Auditing?**

**Auditing** in Azure SQL tracks database events and logs them.
It helps you:

- **Meet compliance needs** (such as ISO, PCI-DSS, GDPR)
- **Analyze database usage patterns**
- **Detect unusual or unauthorized activity**

➡️ Audit logs can be stored in:

- **Azure Storage Account** (for archiving)
- **Log Analytics workspace** (for querying and dashboards)
- **Event Hubs** (for streaming data to other systems)

---

## 🔍 **Server-Level vs Database-Level Auditing**

| Type                        | Scope                                      | Recommended Use                                                 |
| --------------------------- | ------------------------------------------ | --------------------------------------------------------------- |
| **Server-level auditing**   | Applies to **all databases** on the server | Preferred for consistent audit policies                         |
| **Database-level auditing** | Applies to a **specific database only**    | Use only if you need separate logs or settings for one database |

- If **server-level auditing** is enabled, it overrides the database-level audit settings.
- You should **avoid enabling both** unless:

  - You need to send audit logs to a **different storage account/workspace** for a database.
  - You need **different audit rules** for a specific database.

---

## ⚙️ **How to Enable Auditing**

1. Go to **SQL Server > Security > Auditing** in the Azure portal.
2. Turn on auditing.
3. Choose where to store audit logs (**Storage account**, **Log Analytics**, or **Event Hub**).
4. Save your configuration.

---

## 🛡️ **What Events Are Audited by Default?**

Azure SQL includes some pre-configured **Action Groups**, such as:

| Action Group                               | What It Captures                           |
| ------------------------------------------ | ------------------------------------------ |
| `BATCH_COMPLETED_GROUP`                    | All queries and stored procedures executed |
| `SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP` | Successful logins to the database          |
| `FAILED_DATABASE_AUTHENTICATION_GROUP`     | Failed login attempts                      |

You can add other actions as needed to capture more details.

---

## 🔐 **Auditing of Sensitive Data Access**

If you’ve classified your database columns (using **Data Classification**), the audit logs can also capture when sensitive data is accessed.

A new field in the audit logs called **`data_sensitivity_information`** records which sensitivity-labeled columns were accessed.

Example:

- A query runs on a table with a column classified as **"Highly Confidential"**, the audit log shows that sensitive data was involved.

---

## 🔎 **Important Considerations**

- **Auditing on Read-Only Replicas:** Enabled automatically.
- **Performance Impact:** SQL Database optimizes auditing, but under very high loads, some events may not be logged.
- **Retention:** You define how long to keep your audit logs in storage.

---

## ✔️ **Summary**

- Azure SQL auditing tracks login attempts, query executions, and access to sensitive data.
- Recommended setup: **Enable server-level auditing** unless you have special cases.
- Logs help with compliance, troubleshooting, and detecting unusual activity.
- Can be integrated with Azure Monitor for analysis.

---
