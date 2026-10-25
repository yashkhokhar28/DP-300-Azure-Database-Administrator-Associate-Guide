## ✅ **Why Partial Data Migration?**

In some scenarios, you may not want to move the whole database:

- **Security concerns:** e.g., keeping sensitive data like sales data on-premises.
- **Data relevance:** e.g., migrating only active data (e.g., customers & products).
- **Storage & cost optimization:** Avoid migrating unnecessary historical data.
- **Hybrid architecture:** Some data stays on-premises, some goes to Azure.

---

## 📖 **Example Scenario**

In the **bicycle manufacturing** example:

- The on-premises SQL Server contains:

  - **Sales data (sensitive)** → stays on-premises.
  - **Customer & product data (non-sensitive, actively used)** → migrated to Azure SQL Database.

- Purpose: Use **Azure’s scalability** for product/customer data, while maintaining control over sales data.

---

## 🔧 **Data Migration Tools for Partial Migration**

### 1. **bcp Utility (Bulk Copy Program)**

A command-line tool to export and import table data.

- **Export example:**

  ```bash
  bcp AdventureWorks.dbo.Product out product_data.txt -S servername -U username -P password -c
  ```

- **Import example:**

  ```bash
  bcp AzureDB.dbo.Product in product_data.txt -S azureserver.database.windows.net -U username -P password -c
  ```

✔️ Pros:

- Simple, fast for large amounts of data.
- Works for **one table at a time**.
- Can be scripted.

⚠️ Cons:

- Requires careful schema alignment (target table must exist first).
- Doesn't migrate schema, only data.

---

### 2. **Azure Data Factory (ADF)**

A **cloud-based ETL tool** to move and transform data.

✔️ Pros:

- Graphical interface.
- Supports **partial table data**, **filters**, and **transformation logic**.
- Can automate **recurring data migrations**.
- Supports hybrid scenarios (on-prem to Azure).

⚠️ Cons:

- More setup compared to bcp.
- Needs a **Self-hosted Integration Runtime** to connect on-premises SQL Server.

---

## 🛡️ **Summary: Which to Use When?**

| Use Case                                             | Recommended Tool                              |
| ---------------------------------------------------- | --------------------------------------------- |
| Quick, one-time table copy                           | **bcp utility**                               |
| Scheduled, automated, or transformed data migrations | **Azure Data Factory**                        |
| Moving the full schema and data                      | **Azure DMS**, BACPAC/SqlPackage, or transactional replication (Migration extension and DMA are retired) 🔄 |

---
