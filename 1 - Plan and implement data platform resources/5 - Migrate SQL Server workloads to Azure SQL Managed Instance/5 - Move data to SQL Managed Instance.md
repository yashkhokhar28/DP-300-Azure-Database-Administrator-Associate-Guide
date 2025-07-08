## ✅ **Why Synchronize?**

- Often during migration, you **can’t move all users/applications at once**.
- Example: In a **sports retail company**, you migrated the product catalog to Azure SQL MI. Your website now uses Azure, but **other systems are switched over in phases**.
- To keep both databases in sync, you’ll need to **synchronize data regularly**, e.g., hourly.

---

## 🔑 **Connectivity Options Between On-premises and Azure**

To synchronize data, you first need **secure connectivity** between your data center and Azure SQL Managed Instance. Common options:

| Connectivity Type           | Description                                                                                                 |
| --------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Point-to-Site VPN (P2S)** | Secure connection from **a single machine** to Azure, often for testing or admin work.                      |
| **Site-to-Site VPN**        | Secure connection from your **entire network** to Azure for seamless app and database communication.        |
| **ExpressRoute**            | **Private, fast, highly secure** network link between your data center and Azure, ideal for production use. |

---

### 🔓 **Public Endpoint (Optional)**

Azure SQL Managed Instance can expose a **public endpoint (port 3342)** to allow secure internet-based access **without VPN**.

- Requires both a **database admin (to enable it)** and a **network admin (to open firewall port 3342 and configure routing)**.
- Used for scenarios where VPN or ExpressRoute is not possible.

---

## 🔄 **Methods for Data Synchronization**

You have several options depending on your migration or sync needs:

| Method                        | Use Case                                                                   | Direction |
| ----------------------------- | -------------------------------------------------------------------------- | --------- |
| **Native Backup/Restore**     | Full database copy using `.bak` files stored in Azure Blob.                | 1-way     |
| **BACPAC + SqlPackage**       | Metadata + data export, supports smaller, schema-level migrations.         | 1-way     |
| **Bulk Copy Program (BCP)**   | Export/import specific tables as flat files.                               | 1-way     |
| **Azure Data Factory (ADF)**  | Scheduled/automated data moves, transformation, and pipelines.             | 2-way     |
| **Transactional Replication** | **Live replication of table subsets ("articles")**, nearly real-time sync. | 2-way     |

---

## 🔁 **Transactional Replication Highlights**

- **Most common for continuous sync.**
- SQL Managed Instance can act as **publisher**, **distributor**, or **subscriber**.
- Supports **replicating specific tables or parts of tables (called "articles").**
- Works by:

  1. Taking a **snapshot** of the source database.
  2. Replicating **subsequent data changes in real-time**.

- Useful for keeping both on-prem and cloud databases up-to-date during migration.

### ⚙️ **Replication Prerequisites**

- **SQL Authentication** for replication.
- Azure Storage Account used as a **working directory**.
- Required **ports open**:

  - Port 445 (Azure file share access)
  - Port 1433 (if one of the replication roles is on-prem)

---

## 🔐 **How Clients Connect to SQL Managed Instance**

- Managed Instance is placed in a **dedicated subnet** in an Azure virtual network.
- Supports connections via:

  - Azure Portal
  - PowerShell
  - Azure CLI
  - REST API

- **All communication is encrypted** and **verified using certificates**.
- If a certificate is revoked, MI **automatically closes the connection** for security.

---

## ⚙️ **Summary of Synchronization Methods**

| Scenario                               | Recommended Sync Method                     |
| -------------------------------------- | ------------------------------------------- |
| Large database, one-time migration     | Native backup/restore                       |
| Schema + small data export             | BACPAC + SqlPackage                         |
| Table-only migration                   | BCP                                         |
| Scheduled, ETL-style data moves        | Azure Data Factory                          |
| Continuous, real-time table-level sync | **Transactional replication** (recommended) |

---
